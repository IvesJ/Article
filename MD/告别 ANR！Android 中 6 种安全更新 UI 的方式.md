> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/7_hfRNzfPYf5Kyteiiqi4w)

#### 1. **Handler + Looper（经典方式）**

**完整实现（含内存泄漏防护）**：

```
// 使用WeakReference避免Activity内存泄漏
class SafeHandler(activity: Activity) : Handler(Looper.getMainLooper()) {
    private val weakActivity = WeakReference(activity)
    override fun handleMessage(msg: Message) {
        super.handleMessage(msg)
        val activity = weakActivity.get() ?: return
        when (msg.what) {
            MSG_UPDATE_TEXT -> {
                val text = msg.obj as? String
                activity.textView.text = text ?: "Default"
            }
            // 可扩展其他消息类型
        }
    }
    companion object {
        const val MSG_UPDATE_TEXT = 1
    }
}
// 在Activity中使用
class MainActivity : AppCompatActivity() {
    private lateinit var safeHandler: SafeHandler
    private var backgroundThread: Thread? = null
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        safeHandler = SafeHandler(this)
        backgroundThread = Thread {
            // 模拟耗时操作
            val result = heavyWork()
            // 发送消息到主线程
            val msg = safeHandler.obtainMessage(
                SafeHandler.MSG_UPDATE_TEXT, 
                result
            )
            safeHandler.sendMessage(msg)
        }.apply { start() }
    }
    private fun heavyWork(): String {
        Thread.sleep(2000)
        return "Data loaded at ${System.currentTimeMillis()}"
    }
    override fun onDestroy() {
        // 终止后台线程
        backgroundThread?.interrupt()
        safeHandler.removeCallbacksAndMessages(null)
        super.onDestroy()
    }
}

```

**优化点**：

*   增加`WeakReference`防止内存泄漏
    
*   完整生命周期管理（启动 / 销毁）
    
*   支持消息类型扩展
    

#### 2. **View.post()（增强版）**

**带加载状态和错误处理**：

```
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        binding.loadButton.setOnClickListener {
            showLoading(true)
            Thread {
                try {
                    val result = fetchDataFromNetwork()
                    binding.resultText.post {
                        showLoading(false)
                        binding.resultText.text = result
                    }
                } catch (e: Exception) {
                    binding.resultText.post {
                        showLoading(false)
                        showError(e.message ?: "Unknown error")
                    }
                }
            }.start()
        }
    }
    private fun showLoading(show: Boolean) {
        binding.progressBar.visibility = if (show) View.VISIBLE else View.GONE
    }
    private fun showError(message: String) {
        Snackbar.make(binding.root, message, Snackbar.LENGTH_LONG).show()
    }
    @Throws(IOException::class)
    private fun fetchDataFromNetwork(): String {
        // 模拟网络请求
        Thread.sleep(1500)
        if (Random.nextBoolean()) {
            throw IOException("Network unavailable")
        }
        return "Data from server: ${System.currentTimeMillis()}"
    }
}

```

**优化点**：

*   使用 ViewBinding 替代 findViewById
    
*   完整的加载状态管理
    
*   异常捕获和错误展示
    
*   线程安全的数据传递
    

#### 3. **Activity.runOnUiThread（完整流程）**

**含数据缓存和重试机制**：

```
class MainActivity : AppCompatActivity() {
    private var isLoading = false
    private var lastData: String? = null
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val refreshButton: Button = findViewById(R.id.refreshButton)
        val dataTextView: TextView = findViewById(R.id.dataTextView)
        // 恢复缓存数据
        lastData?.let { dataTextView.text = it }
        refreshButton.setOnClickListener {
            if (isLoading) return@setOnClickListener
            isLoading = true
            dataTextView.text = "Loading..."
            Thread {
                try {
                    val newData = loadData()
                    runOnUiThread {
                        isLoading = false
                        lastData = newData
                        dataTextView.text = newData
                    }
                } catch (e: Exception) {
                    runOnUiThread {
                        isLoading = false
                        showRetryDialog(e)
                    }
                }
            }.start()
        }
    }
    private fun showRetryDialog(e: Exception) {
        AlertDialog.Builder(this)
            .setTitle("Error")
            .setMessage(e.message ?: "Failed to load data")
            .setPositiveButton("Retry") { _, _ ->
                // 重新触发加载
                findViewById<Button>(R.id.refreshButton).performClick()
            }
            .setNegativeButton("Cancel", null)
            .show()
    }
    private fun loadData(): String {
        // 模拟可能失败的操作
        Thread.sleep(1000)
        if (Random.nextInt(10) > 7) {
            throw RuntimeException("Data source error")
        }
        return "New data: ${System.currentTimeMillis()}"
    }
}

```

**优化点**：

*   加载状态锁定防止重复请求
    
*   数据缓存和状态恢复
    
*   完整的错误恢复机制
    
*   UI 状态自动管理
    

#### 4. **Kotlin 协程（完整 MVVM 实现）**

**使用 ViewModel + Repository 模式**：

```
// Repository层
class DataRepository {
    suspend fun fetchData(): String {
        delay(1000) // 模拟网络延迟
        if (Random.nextBoolean()) {
            throw RuntimeException("Server error")
        }
        return "Coroutine data: ${System.currentTimeMillis()}"
    }
}
// ViewModel
class MainViewModel : ViewModel() {
    private val repository = DataRepository()
    private val _uiState = MutableStateFlow<UiState>(UiState.Idle)
    val uiState: StateFlow<UiState> = _uiState
    fun loadData() {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            try {
                val data = repository.fetchData()
                _uiState.value = UiState.Success(data)
            } catch (e: Exception) {
                _uiState.value = UiState.Error(e.message ?: "Unknown error")
            }
        }
    }
    sealed class UiState {
        object Idle : UiState()
        object Loading : UiState()
        data class Success(val data: String) : UiState()
        data class Error(val message: String) : UiState()
    }
}
// Activity
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private val viewModel: MainViewModel by viewModels()
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        setupObservers()
        binding.loadButton.setOnClickListener { viewModel.loadData() }
    }
    private fun setupObservers() {
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { state ->
                    when (state) {
                        is MainViewModel.UiState.Idle -> resetUI()
                        is MainViewModel.UiState.Loading -> showLoading()
                        is MainViewModel.UiState.Success -> showData(state.data)
                        is MainViewModel.UiState.Error -> showError(state.message)
                    }
                }
            }
        }
    }
    private fun showLoading() {
        binding.progressBar.visibility = View.VISIBLE
        binding.dataTextView.text = "Loading..."
    }
    private fun showData(data: String) {
        binding.progressBar.visibility = View.GONE
        binding.dataTextView.text = data
    }
    private fun showError(message: String) {
        binding.progressBar.visibility = View.GONE
        Snackbar.make(binding.root, message, Snackbar.LENGTH_LONG)
            .setAction("Retry") { viewModel.loadData() }
            .show()
    }
    private fun resetUI() {
        binding.progressBar.visibility = View.GONE
        binding.dataTextView.text = "Click to load"
    }
}

```

**优化点**：

*   完整的 MVVM 架构实现
    
*   使用 StateFlow 管理 UI 状态
    
*   生命周期感知的数据收集
    
*   全面的状态处理（加载 / 成功 / 错误 / 空闲）
    

#### 5. **LiveData + ViewModel（高级实现）**

**带数据转换和 MediatorLiveData**：

```
class AdvancedViewModel : ViewModel() {
    private val repository = DataRepository()
    private val _rawData = MutableLiveData<String>()
    private val _error = MutableLiveData<String>()
    val processedData: LiveData<String> = Transformations.map(_rawData) {
        "Processed: $it" // 数据转换示例
    }
    val uiState: MediatorLiveData<UiState> = MediatorLiveData<UiState>().apply {
        addSource(_rawData) { data ->
            value = if (data != null) UiState.Success(data) else UiState.Loading
        }
        addSource(_error) { error ->
            error?.let { value = UiState.Error(it) }
        }
    }
    fun fetchData() {
        uiState.value = UiState.Loading
        viewModelScope.launch(Dispatchers.IO) {
            try {
                val data = repository.fetchData()
                _rawData.postValue(data)
            } catch (e: Exception) {
                _error.postValue(e.message ?: "Unknown error")
            }
        }
    }
    sealed class UiState {
        object Loading : UiState()
        data class Success(val data: String) : UiState()
        data class Error(val message: String) : UiState()
    }
}
// 在Fragment中使用
class DataFragment : Fragment() {
    private lateinit var binding: FragmentDataBinding
    private val viewModel: AdvancedViewModel by viewModels()
    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View {
        binding = FragmentDataBinding.inflate(inflater, container, false)
        return binding.root
    }
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        viewModel.uiState.observe(viewLifecycleOwner) { state ->
            when (state) {
                is AdvancedViewModel.UiState.Loading -> {
                    binding.progress.visibility = View.VISIBLE
                    binding.resultText.text = "Loading..."
                }
                is AdvancedViewModel.UiState.Success -> {
                    binding.progress.visibility = View.GONE
                    binding.resultText.text = state.data
                }
                is AdvancedViewModel.UiState.Error -> {
                    binding.progress.visibility = View.GONE
                    binding.resultText.text = "Error: ${state.message}"
                }
            }
        }
        viewModel.processedData.observe(viewLifecycleOwner) { data ->
            binding.processedText.text = data
        }
        binding.refreshButton.setOnClickListener {
            viewModel.fetchData()
        }
    }
}

```

**优化点**：

*   使用 Transformations 进行数据加工
    
*   MediatorLiveData 组合多个数据源
    
*   Fragment 中的安全观察
    
*   分离原始数据和展示数据
    

#### 6. **RxJava（完整链式调用）**

**含超时控制和重试机制**：

```
class RxJavaExampleActivity : AppCompatActivity() {
    private lateinit var binding: ActivityRxjavaBinding
    private val compositeDisposable = CompositeDisposable()
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityRxjavaBinding.inflate(layoutInflater)
        setContentView(binding.root)
        binding.startButton.setOnClickListener {
            startRxOperation()
        }
    }
    private fun startRxOperation() {
        val disposable = Single.fromCallable { 
            // 模拟可能失败的操作
            if (Random.nextBoolean()) {
                throw IOException("Simulated network error")
            }
            "RxJava data: ${System.currentTimeMillis()}"
        }
        .subscribeOn(Schedulers.io())
        .timeout(2, TimeUnit.SECONDS)
        .retryWhen { errors ->
            errors.flatMap { error ->
                if (error is TimeoutException) {
                    showRetryNotification("Timeout, retrying...")
                    Flowable.timer(1, TimeUnit.SECONDS)
                } else {
                    Flowable.error(error)
                }
            }
        }
        .observeOn(AndroidSchedulers.mainThread())
        .doOnSubscribe { showLoading(true) }
        .doFinally { showLoading(false) }
        .subscribe(
            { result ->
                binding.resultText.text = result
                binding.resultText.setTextColor(Color.GREEN)
            },
            { error ->
                binding.resultText.text = "Error: ${error.message}"
                binding.resultText.setTextColor(Color.RED)
            }
        )
        compositeDisposable.add(disposable)
    }
    private fun showLoading(show: Boolean) {
        binding.progressBar.visibility = if (show) View.VISIBLE else View.GONE
    }
    private fun showRetryNotification(message: String) {
        runOnUiThread {
            Toast.makeText(this, message, Toast.LENGTH_SHORT).show()
        }
    }
    override fun onDestroy() {
        compositeDisposable.clear()
        super.onDestroy()
    }
}

```

**优化点**：

*   完整的 RxJava 操作链
    
*   超时控制和智能重试
    
*   资源自动管理（CompositeDisposable）
    
*   副作用处理（doOnSubscribe/doFinally）
    
*   线程切换和错误处理分离
    

### 如何选择合适的方式？

*   **简单场景**：优先选择`View.post()`或`runOnUiThread`。
    
*   **协程项目**：使用`Dispatchers.Main`或`LiveData`+`ViewModel`。
    
*   **响应式编程**：考虑 RxJava。
    
*   **历史代码维护**：可保留 Handler，但需注意内存泄漏。
    

### 注意事项

1.  **避免内存泄漏**：在组件销毁时取消异步任务（如协程的`lifecycleScope`自动管理）。
    
2.  **线程安全**：确保非 UI 线程不直接操作 View。
    
3.  **ANR 阈值**：主线程阻塞超过 5 秒触发 ANR，务必缩短主线程耗时操作。
    

通过合理选择线程切换策略，结合 Android 最新架构组件，可以彻底告别 ANR 问题，实现流畅的 UI 体验！