# RustChatGPT项目源代码



# cmd.rs

```rust
// 从 tauri 库中导入必要的模块
use tauri::{command, AppHandle, LogicalPosition, Manager, PhysicalSize};

// 从项目中导入自定义模块和常量
use crate::core::{
    conf::AppConf,
    constant::{ASK_HEIGHT, TITLEBAR_HEIGHT},
};

// 定义一个命令来重新加载视图
#[command]
pub fn view_reload(app: AppHandle) {
    app.get_window("core") // 获取名为 "core" 的窗口
        .unwrap() // 解包结果，如果有错误则抛出 panic
        .get_webview("main") // 获取名为 "main" 的 webview
        .unwrap()
        .eval("window.location.reload()") // 执行 JavaScript 代码重新加载页面
        .unwrap();
}

// 定义一个命令来获取当前视图的 URL
#[command]
pub fn view_url(app: AppHandle) -> tauri::Url {
    app.get_window("core") // 获取名为 "core" 的窗口
        .unwrap()
        .get_webview("main") // 获取名为 "main" 的 webview
        .unwrap()
        .url() // 获取当前 URL
        .unwrap()
}

// 定义一个命令来前进到历史记录中的下一页
#[command]
pub fn view_go_forward(app: AppHandle) {
    app.get_window("core") // 获取名为 "core" 的窗口
        .unwrap()
        .get_webview("main") // 获取名为 "main" 的 webview
        .unwrap()
        .eval("window.history.forward()") // 执行 JavaScript 代码前进历史记录
        .unwrap();
}

// 定义一个命令来后退到历史记录中的上一页
#[command]
pub fn view_go_back(app: AppHandle) {
    app.get_window("core") // 获取名为 "core" 的窗口
        .unwrap()
        .get_webview("main") // 获取名为 "main" 的 webview
        .unwrap()
        .eval("window.history.back()") // 执行 JavaScript 代码后退历史记录
        .unwrap();
}

// 定义一个命令来固定或取消固定窗口
#[command]
pub fn window_pin(app: AppHandle, pin: bool) {
    let conf = AppConf::load(&app).unwrap(); // 加载应用程序配置
    conf.amend(serde_json::json!({"stay_on_top": pin})) // 修改配置以设置是否固定窗口
        .unwrap()
        .save(&app) // 保存配置
        .unwrap();

    app.get_window("core") // 获取名为 "core" 的窗口
        .unwrap()
        .set_always_on_top(pin) // 设置窗口是否总是在最前
        .unwrap();
}

// 定义一个命令来同步 Ask 模式的信息
#[command]
pub fn ask_sync(app: AppHandle, message: String) {
    app.get_window("core") // 获取名为 "core" 的窗口
        .unwrap()
        .get_webview("main") // 获取名为 "main" 的 webview
        .unwrap()
        .eval(&format!("ChatAsk.sync({})", message)) // 执行 JavaScript 代码同步消息
        .unwrap();
}

// 定义一个命令来发送 Ask 模式的信息
#[command]
pub fn ask_send(app: AppHandle) {
    let win = app.get_window("core").unwrap(); // 获取名为 "core" 的窗口

    win.get_webview("main") // 获取名为 "main" 的 webview
        .unwrap()
        .eval(r#"
        ChatAsk.submit(); // 提交 Ask 模式表单
        setTimeout(() => {
            __TAURI__.webview.Webview.getByLabel('ask')?.setFocus(); // 设置 Ask 模式的 webview 为焦点
        }, 500);
        "#)
        .unwrap();
}

// 定义一个命令来设置应用程序的主题
#[command]
pub fn set_theme(app: AppHandle, theme: String) {
    let conf = AppConf::load(&app).unwrap(); // 加载应用程序配置
    conf.amend(serde_json::json!({"theme": theme})) // 修改配置以设置主题
        .unwrap()
        .save(&app) // 保存配置
        .unwrap();

    app.restart(); // 重启应用程序
}

// 定义一个命令来获取应用程序的配置
#[command]
pub fn get_app_conf(app: AppHandle) -> AppConf {
    AppConf::load(&app).unwrap() // 加载并返回应用程序配置
}

// 定义一个命令来设置 Ask 模式视图的启用状态
#[command]
pub fn set_view_ask(app: AppHandle, enabled: bool) {
    let conf = AppConf::load(&app).unwrap(); // 加载应用程序配置
    conf.amend(serde_json::json!({"ask_mode": enabled})) // 修改配置以启用或禁用 Ask 模式
        .unwrap()
        .save(&app) // 保存配置
        .unwrap();

    let core_window = app.get_window("core").unwrap(); // 获取名为 "core" 的窗口
    let ask_mode_height = if enabled { ASK_HEIGHT } else { 0.0 }; // 根据启用状态设置 Ask 模式的高度
    let scale_factor = core_window.scale_factor().unwrap(); // 获取窗口的缩放因子
    let titlebar_height = (scale_factor * TITLEBAR_HEIGHT).round() as u32; // 计算标题栏高度
    let win_size = core_window
        .inner_size() // 获取窗口内部大小
        .unwrap();
    let ask_height = (scale_factor * ask_mode_height).round() as u32; // 计算 Ask 模式的高度

    let main_view = core_window
        .get_webview("main") // 获取名为 "main" 的 webview
        .unwrap();
    let titlebar_view = core_window
        .get_webview("titlebar") // 获取名为 "titlebar" 的 webview
        .unwrap();
    let ask_view = core_window
        .get_webview("ask") // 获取名为 "ask" 的 webview
        .unwrap();

    if enabled {
        ask_view.set_focus().unwrap(); // 如果启用，设置 Ask 模式的 webview 为焦点
    } else {
        main_view.set_focus().unwrap(); // 否则，设置 main webview 为焦点
    }

    let set_view_properties =
        |view: &tauri::Webview, position: LogicalPosition<f64>, size: PhysicalSize<u32>| {
            if let Err(e) = view.set_position(position) {
                eprintln!("[cmd:view:position] 设置视图位置失败: {}", e); // 设置视图位置失败时打印错误信息
            }
            if let Err(e) = view.set_size(size) {
                eprintln!("[cmd:view:size] 设置视图大小失败: {}", e); // 设置视图大小失败时打印错误信息
            }
        };

    #[cfg(target_os = "macos")]
    {
        set_view_properties(
            &main_view,
            LogicalPosition::new(0.0, TITLEBAR_HEIGHT),
            PhysicalSize::new(
                win_size.width,
                win_size.height - (titlebar_height + ask_height),
            ),
        );
        set_view_properties(
            &titlebar_view,
            LogicalPosition::new(0.0, 0.0),
            PhysicalSize::new(win_size.width, titlebar_height),
        );
        set_view_properties(
            &ask_view,
            LogicalPosition::new(
                0.0,
                (win_size.height as f64 / scale_factor) - ask_mode_height,
            ),
            PhysicalSize::new(win_size.width, ask_height),
        );
    }

    #[cfg(not(target_os = "macos"))]
    {
        set_view_properties(
            &main_view,
            LogicalPosition::new(0.0, 0.0),
            PhysicalSize::new(
                win_size.width,
                win_size.height - (ask_height + titlebar_height),
            ),
        );
        set_view_properties(
            &titlebar_view,
            LogicalPosition::new(
                0.0,
                (win_size.height as f64 / scale_factor) - TITLEBAR_HEIGHT,
            ),
            PhysicalSize::new(win_size.width, titlebar_height),
        );
        set_view_properties(
            &ask_view,
            LogicalPosition::new(
                0.0,
                (win_size.height as f64 / scale_factor) - ask_mode_height - TITLEBAR_HEIGHT,
            ),
            PhysicalSize::new(win_size.width, ask_height),
        );
    }
}

```



# conf.rs

```rust
// 导入必要的库和模块
use log::error;
use serde::{Deserialize, Serialize};
use serde_json::Value;
use std::{
    collections::BTreeMap,
    fs::{self, File},
    io::{Read, Write},
    path::PathBuf,
};
use tauri::{AppHandle, Manager, Theme};

// 定义应用程序配置的结构体
#[derive(Serialize, Deserialize, Debug)]
pub struct AppConf {
    pub theme: String,              // 主题
    pub stay_on_top: bool,          // 窗口是否总在最前
    pub ask_mode: bool,             // Ask 模式是否启用
    pub mac_titlebar_hidden: bool,  // macOS 标题栏是否隐藏
}

// 为 AppConf 结构体实现方法
impl AppConf {
    // 创建一个新的配置对象
    pub fn new() -> Self {
        Self {
            theme: "system".to_string(), // 默认主题是系统主题
            stay_on_top: false,          // 默认窗口不总在最前
            ask_mode: false,             // 默认不启用 Ask 模式
            #[cfg(target_os = "macos")]
            mac_titlebar_hidden: true,   // 在 macOS 上默认隐藏标题栏
            #[cfg(not(target_os = "macos"))]
            mac_titlebar_hidden: false,  // 在其他系统上默认不隐藏标题栏
        }
    }

    // 获取配置文件路径
    pub fn get_conf_path(app: &AppHandle) -> Result<PathBuf, Box<dyn std::error::Error>> {
        let config_dir = app
            .path()
            .config_dir()? // 获取配置目录
            .join("com.nofwl.chatgpt") // 添加子目录
            .join("config.json"); // 配置文件名
        Ok(config_dir)
    }

    // 获取脚本文件夹路径
    pub fn get_scripts_path(app: &AppHandle) -> Result<PathBuf, Box<dyn std::error::Error>> {
        let scripts_dir = app
            .path()
            .config_dir()? // 获取配置目录
            .join("com.nofwl.chatgpt") // 添加子目录
            .join("scripts"); // 脚本文件夹名
        Ok(scripts_dir)
    }

    // 加载脚本文件的内容
    pub fn load_script(app: &AppHandle, filename: &str) -> String {
        let script_file = Self::get_scripts_path(app).unwrap().join(filename); // 获取脚本文件路径
        fs::read_to_string(script_file).unwrap_or_else(|_| "".to_string()) // 读取文件内容，出错则返回空字符串
    }

    // 加载配置文件
    pub fn load(app: &AppHandle) -> Result<Self, Box<dyn std::error::Error>> {
        let path = Self::get_conf_path(app)?; // 获取配置文件路径

        if !path.exists() {
            let config = Self::new(); // 如果文件不存在，创建新的配置对象
            config.save(app)?; // 保存新的配置文件
            return Ok(config); // 返回新的配置对象
        }

        let mut file = File::open(path)?; // 打开配置文件
        let mut contents = String::new();
        file.read_to_string(&mut contents)?; // 读取文件内容
        let config: Result<AppConf, _> = serde_json::from_str(&contents); // 解析 JSON 内容为 AppConf 对象

        // 处理条件字段并在必要时回退到默认值
        if let Err(e) = &config {
            error!("[conf::load] {}", e); // 记录错误日志
            let mut default_config = Self::new();
            default_config = default_config.amend(serde_json::from_str(&contents)?)?; // 合并默认配置和当前内容
            default_config.save(app)?; // 保存新的配置文件
            return Ok(default_config); // 返回新的配置对象
        }

        Ok(config?) // 返回加载的配置对象
    }

    // 保存配置文件
    pub fn save(&self, app: &AppHandle) -> Result<(), Box<dyn std::error::Error>> {
        let path = Self::get_conf_path(app)?; // 获取配置文件路径

        if let Some(dir) = path.parent() {
            fs::create_dir_all(dir)?; // 创建所有必要的目录
        }

        let mut file = File::create(path)?; // 创建配置文件
        let contents = serde_json::to_string_pretty(self)?; // 将配置对象转换为格式化的 JSON 字符串
        // dbg!(&contents);
        file.write_all(contents.as_bytes())?; // 将 JSON 字符串写入文件
        Ok(())
    }

    // 修改配置
    pub fn amend(self, json: Value) -> Result<Self, serde_json::Error> {
        let val = serde_json::to_value(self)?; // 将当前配置对象转换为 JSON 值
        let mut config: BTreeMap<String, Value> = serde_json::from_value(val)?; // 将 JSON 值转换为 BTreeMap
        let new_json: BTreeMap<String, Value> = serde_json::from_value(json)?; // 将新的 JSON 值转换为 BTreeMap

        for (k, v) in new_json {
            config.insert(k, v); // 合并新的配置项
        }

        let config_str = serde_json::to_string_pretty(&config)?; // 将合并后的配置转换为格式化的 JSON 字符串
        serde_json::from_str::<AppConf>(&config_str).map_err(|err| { // 将 JSON 字符串解析为 AppConf 对象
            error!("[conf::amend] {}", err); // 记录错误日志
            err
        })
    }

    // 获取当前主题
    pub fn get_theme(app: &AppHandle) -> Theme {
        let theme = Self::load(app).unwrap().theme; // 加载配置并获取主题
        match theme.as_str() {
            "system" => match dark_light::detect() {
                dark_light::Mode::Dark => Theme::Dark, // 系统模式为暗时，设置为暗主题
                dark_light::Mode::Light => Theme::Light, // 系统模式为亮时，设置为亮主题
                dark_light::Mode::Default => Theme::Light, // 系统模式默认时，设置为亮主题
            },
            "dark" => Theme::Dark, // 配置为暗主题时，设置为暗主题
            _ => Theme::Light, // 其他情况，设置为亮主题
        }
    }
}

```



# constant.rs

```rust
// 定义标题栏高度为 28.0 像素
pub static TITLEBAR_HEIGHT: f64 = 28.0;

// 定义 ASK 窗口的高度为 120.0 像素
pub static ASK_HEIGHT: f64 = 120.0;

// 定义一个初始化脚本，当网页内容加载完成后会执行此脚本
pub static INIT_SCRIPT: &str = r#"
window.addEventListener('DOMContentLoaded', function() {
    // 定义一个处理 URL 变化的函数
    function handleUrlChange() {
        const url = window.location.href;
        if (url !== 'about:blank') {
            console.log('URL changed:', url);
            window.__TAURI__.webviewWindow.WebviewWindow.getByLabel('titlebar').emit('navigation:change', { url });
        }
    }

    // 定义一个处理链接点击的函数
    function handleLinkClick(event) {
        const target = event.target;
        // 如果点击的是一个链接，并且链接的 target 属性不是 '_blank'，则将其改为 '_blank'
        if (target.tagName === 'A' && target.target && target.target !== '_blank') {
            target.target = '_blank';
        }
    }

    // 添加事件监听器，当点击事件发生时调用 handleLinkClick 函数
    document.addEventListener('click', handleLinkClick, true);

    // 添加事件监听器，当浏览器历史状态变化时调用 handleUrlChange 函数
    window.addEventListener('popstate', handleUrlChange);
    window.addEventListener('pushState', handleUrlChange);
    window.addEventListener('replaceState', handleUrlChange);

    // 保存原始的 pushState 和 replaceState 方法
    const originalPushState = history.pushState;
    const originalReplaceState = history.replaceState;

    // 重写 pushState 方法，在调用原始方法后调用 handleUrlChange 函数
    history.pushState = function() {
        originalPushState.apply(this, arguments);
        console.log('pushState called');
        handleUrlChange();
    };

    // 重写 replaceState 方法，在调用原始方法后调用 handleUrlChange 函数
    history.replaceState = function() {
        originalReplaceState.apply(this, arguments);
        console.log('replaceState called');
        handleUrlChange();
    };

    // 初始化时调用 handleUrlChange 函数，以便检查初始 URL
    handleUrlChange();
});
"#;

```



# mod.rs

```rust
// 声明一个名为 cmd 的模块，该模块通常用于定义命令或函数，这些命令或函数可以在 Tauri 应用程序中被调用
pub mod cmd;

// 声明一个名为 conf 的模块，该模块通常用于处理应用程序的配置文件
pub mod conf;

// 声明一个名为 constant 的模块，该模块通常用于定义常量
pub mod constant;

// 声明一个名为 setup 的模块，该模块通常用于应用程序的初始化设置
pub mod setup;

// 声明一个名为 template 的模块，该模块通常用于处理模板相关的功能
pub mod template;

```



# setup.rs

```rust
// 引入所需的标准库模块
use std::{
    path::PathBuf,
    sync::{Arc, Mutex},
};

// 引入 Tauri 库中的模块和结构体
use tauri::{
    webview::DownloadEvent, App, LogicalPosition, Manager, PhysicalSize, WebviewBuilder,
    WebviewUrl, WindowBuilder, WindowEvent,
};

// 引入 Tauri 插件扩展模块
use tauri_plugin_shell::ShellExt;

// 仅在 macOS 系统上引入 TitleBarStyle 模块
#[cfg(target_os = "macos")]
use tauri::TitleBarStyle;

// 引入当前 crate 中的模块和常量
use crate::core::{
    conf::AppConf,
    constant::{ASK_HEIGHT, INIT_SCRIPT, TITLEBAR_HEIGHT},
    template,
};

// 初始化函数，接受一个 Tauri 应用程序的可变引用作为参数
pub fn init(app: &mut App) -> Result<(), Box<dyn std::error::Error>> {
    let handle = app.handle();

    // 加载应用程序配置
    let conf = &AppConf::load(handle)?;
    let ask_mode_height = if conf.ask_mode { ASK_HEIGHT } else { 0.0 };

    // 初始化模板引擎，加载脚本路径
    template::Template::new(AppConf::get_scripts_path(handle)?);

    // 异步运行块，处理窗口创建和视图初始化
    tauri::async_runtime::spawn({
        let handle = handle.clone();
        async move {
            // 创建核心窗口
            let mut core_window = WindowBuilder::new(&handle, "core").title("ChatGPT");

            #[cfg(target_os = "macos")]
            {
                // 设置 macOS 特有的标题栏样式和隐藏标题
                core_window = core_window
                    .title_bar_style(TitleBarStyle::Overlay)
                    .hidden_title(true);
            }

            // 设置窗口大小、可调整大小和主题
            core_window = core_window
                .resizable(true)
                .inner_size(800.0, 600.0)
                .min_inner_size(300.0, 200.0)
                .theme(Some(AppConf::get_theme(&handle)));

            // 构建窗口并获取窗口尺寸
            let core_window = core_window
                .build()
                .expect("[core:window] Failed to build window");
            let win_size = core_window
                .inner_size()
                .expect("[core:window] Failed to get window size");

            // 将窗口包装在 Arc<Mutex<_>> 中，以便在多个线程之间管理所有权
            let window = Arc::new(Mutex::new(core_window));

            // 创建主视图，并设置下载事件处理器
            let main_view =
                WebviewBuilder::new("main", WebviewUrl::App("https://chatgpt.com".into()))
                    .auto_resize()
                    .on_download({
                        let app_handle = handle.clone();
                        let download_path = Arc::new(Mutex::new(PathBuf::new()));
                        move |_, event| {
                            match event {
                                DownloadEvent::Requested { destination, .. } => {
                                    let download_dir = app_handle
                                        .path()
                                        .download_dir()
                                        .expect("[view:download] Failed to get download directory");
                                    let mut locked_path = download_path
                                        .lock()
                                        .expect("[view:download] Failed to lock download path");
                                    *locked_path = download_dir.join(&destination);
                                    *destination = locked_path.clone();
                                }
                                DownloadEvent::Finished { success, .. } => {
                                    let final_path = download_path
                                        .lock()
                                        .expect("[view:download] Failed to lock download path")
                                        .clone();

                                    if success {
                                        app_handle
                                            .shell()
                                            .open(final_path.to_string_lossy(), None)
                                            .expect("[view:download] Failed to open file");
                                    }
                                }
                                _ => (),
                            }
                            true
                        }
                    })
                    .initialization_script(&AppConf::load_script(&handle, "ask.js"))
                    .initialization_script(INIT_SCRIPT);

            // 创建标题栏视图和 ASK 视图
            let titlebar_view = WebviewBuilder::new(
                "titlebar",
                WebviewUrl::App("index.html?type=titlebar".into()),
            )
            .auto_resize();

            let ask_view =
                WebviewBuilder::new("ask", WebviewUrl::App("index.html?type=ask".into()))
                    .auto_resize();

            // 获取窗口的缩放因子和计算标题栏和 ASK 的高度
            let win = window.lock().unwrap();
            let scale_factor = win.scale_factor().unwrap();
            let titlebar_height = (scale_factor * TITLEBAR_HEIGHT).round() as u32;
            let ask_height = (scale_factor * ask_mode_height).round() as u32;

            #[cfg(target_os = "macos")]
            {
                // macOS 系统上的布局设置
                let main_area_height = win_size.height - titlebar_height;
                win.add_child(
                    titlebar_view,
                    LogicalPosition::new(0, 0),
                    PhysicalSize::new(win_size.width, titlebar_height),
                )
                .unwrap();
                win.add_child(
                    ask_view,
                    LogicalPosition::new(
                        0.0,
                        (win_size.height as f64 / scale_factor) - ask_mode_height,
                    ),
                    PhysicalSize::new(win_size.width, ask_height),
                )
                .unwrap();
                win.add_child(
                    main_view,
                    LogicalPosition::new(0.0, TITLEBAR_HEIGHT),
                    PhysicalSize::new(win_size.width, main_area_height - ask_height),
                )
                .unwrap();
            }

            #[cfg(not(target_os = "macos"))]
            {
                // 非 macOS 系统上的布局设置
                win.add_child(
                    ask_view,
                    LogicalPosition::new(
                        0.0,
                        (win_size.height as f64 / scale_factor) - ask_mode_height,
                    ),
                    PhysicalSize::new(win_size.width, ask_height),
                )
                .unwrap();
                win.add_child(
                    titlebar_view,
                    LogicalPosition::new(
                        0.0,
                        (win_size.height as f64 / scale_factor) - ask_mode_height - TITLEBAR_HEIGHT,
                    ),
                    PhysicalSize::new(win_size.width, titlebar_height),
                )
                .unwrap();
                win.add_child(
                    main_view,
                    LogicalPosition::new(0.0, 0.0),
                    PhysicalSize::new(
                        win_size.width,
                        win_size.height - (ask_height + titlebar_height),
                    ),
                )
                .unwrap();
            }

            // 创建视图属性设置函数
            let window_clone = Arc::clone(&window);
            let set_view_properties =
                |view: &tauri::Webview, position: LogicalPosition<f64>, size: PhysicalSize<u32>| {
                    if let Err(e) = view.set_position(position) {
                        eprintln!("[view:position] Failed to set view position: {}", e);
                    }
                    if let Err(e) = view.set_size(size) {
                        eprintln!("[view:size] Failed to set view size: {}", e);
                    }
                };

            // 监听窗口事件，调整视图布局
            win.on_window_event(move |event| {
                let conf = &AppConf::load(&handle).unwrap();
                let ask_mode_height = if conf.ask_mode { ASK_HEIGHT } else { 0.0 };
                let ask_height = (scale_factor * ask_mode_height).round() as u32;

                if let WindowEvent::Resized(size) = event {
                    let win = window_clone.lock().unwrap();

                    let main_view = win
                        .get_webview("main")
                        .expect("[view:main] Failed to get webview window");
                    let titlebar_view = win
                        .get_webview("titlebar")
                        .expect("[view:titlebar] Failed to get webview window");
                    let ask_view = win
                        .get_webview("ask")
                        .expect("[view:ask] Failed to get webview window");

                    #[cfg(target_os = "macos")]
                    {
                        set_view_properties(
                            &main_view,
                            LogicalPosition::new(0.0, TITLEBAR_HEIGHT),
                            PhysicalSize::new(
                                size.width,
                                size.height - (titlebar_height + ask_height),
                            ),
                        );
                        set_view_properties(
                            &titlebar_view,
                            LogicalPosition::new(0.0, 0.0),
                            PhysicalSize::new(size.width, titlebar_height),
                        );
                        set_view_properties(
                            &ask_view,
                            LogicalPosition::new(
                                0.0,
                                (size.height as f64 / scale_factor) - ask_mode_height,
                            ),
                            PhysicalSize::new(size.width, ask_height),
                        );
                    }

                    #[cfg(not(target_os = "macos"))]
                    {
                        set_view_properties(
                            &main_view,
                            LogicalPosition::new(0.0, 0.0),
                            PhysicalSize::new(
                                size.width,
                                size.height - (ask_height + titlebar_height),
                            ),
                        );
                        set_view_properties(
                            &titlebar_view,
                            LogicalPosition::new(
                                0.0,
                                (size.height as f64 / scale_factor) - TITLEBAR_HEIGHT,
                            ),
                            PhysicalSize::new(size.width, titlebar_height),
                        );
                        set_view_properties(
                            &ask_view,
                            LogicalPosition::new(
                                0.0,
                                (size.height as f64 / scale_factor)
                                    - ask_mode_height
                                    - TITLEBAR_HEIGHT,
                            ),
                            PhysicalSize::new(size.width, ask_height),
                        );
                    }
                }
            });
        }
    });

    Ok(())
}

```



# template.rs

```rust
// 引入所需的外部库
use anyhow::{Context, Result};
use log::{error, info};
use regex::Regex;
use semver::Version;
use serde_json::json;
use std::{
    fs::{self, File},
    io::{Read, Write},
    path::Path,
};

// 包含脚本数据的静态变量
pub static SCRIPT_ASK: &[u8] = include_bytes!("../../scripts/ask.js");

/// 表示包含脚本数据的模板结构体。
#[derive(Debug)]
pub struct Template {
    pub ask: Vec<u8>,
}

impl Template {
    /// 创建一个新的 Template 实例，并用脚本数据初始化它。
    pub fn new<P: AsRef<Path>>(template_dir: P) -> Self {
        let template_dir = template_dir.as_ref();
        let mut template = Template::default();

        let files = vec![(template_dir.join("ask.js"), &mut template.ask)];

        for (filename, _) in files {
            match update_or_create_file(&filename, SCRIPT_ASK) {
                Ok(updated) => {
                    if updated {
                        info!("脚本已更新或创建: {}", filename.display());
                    } else {
                        info!("脚本是最新的: {}", filename.display());
                    }
                }
                Err(e) => {
                    error!("处理脚本失败, {}: {}", filename.display(), e);
                }
            }
        }

        template
    }
}

impl Default for Template {
    fn default() -> Template {
        Template {
            ask: Vec::from(SCRIPT_ASK),
        }
    }
}

/// 从给定数据中读取版本信息。
fn read_version_info(data: &[u8]) -> Result<serde_json::Value> {
    let content = String::from_utf8_lossy(data);
    let re_name = Regex::new(r"@name\s+(.*?)\n").context("编译名称正则表达式失败")?;
    let re_version =
        Regex::new(r"@version\s+(.*?)\n").context("编译版本正则表达式失败")?;
    let re_url = Regex::new(r"@url\s+(.*?)\n").context("编译URL正则表达式失败")?;

    let name = re_name
        .captures(&content)
        .and_then(|cap| cap.get(1))
        .map_or(String::new(), |m| m.as_str().trim().to_string());

    let version = re_version
        .captures(&content)
        .and_then(|cap| cap.get(1))
        .map_or(String::new(), |m| m.as_str().trim().to_string());

    let url = re_url
        .captures(&content)
        .and_then(|cap| cap.get(1))
        .map_or(String::new(), |m| m.as_str().trim().to_string());

    let json_data = json!({
        "name": name,
        "version": version,
        "url": url,
    });

    Ok(json_data)
}

/// 读取给定文件的内容。
fn read_file_contents<P: AsRef<Path>>(filename: P) -> Result<Vec<u8>> {
    let filename = filename.as_ref();
    let mut file = File::open(filename)?;
    let mut contents = Vec::new();
    file.read_to_end(&mut contents)?;
    Ok(contents)
}

/// 将给定数据写入指定文件。
fn write_file_contents<P: AsRef<Path>>(filename: P, data: &[u8]) -> Result<()> {
    let filename = filename.as_ref();
    let mut file = File::create(filename)?;
    file.write_all(data)?;
    Ok(())
}

/// 为指定文件路径创建必要的目录。
fn create_dir<P: AsRef<Path>>(filename: P) -> Result<()> {
    let filename = filename.as_ref();
    if let Some(parent) = filename.parent() {
        if !parent.exists() {
            fs::create_dir_all(parent)?;
        }
    }
    Ok(())
}

/// 如果新数据的版本较新或版本信息缺失，则更新文件；
/// 或者，如果文件不存在，则创建文件。
fn update_or_create_file<P: AsRef<Path>>(filename: P, new_data: &[u8]) -> Result<bool> {
    let filename = filename.as_ref();

    // 确保目录存在
    create_dir(filename)?;

    let current_data = read_file_contents(filename);

    match current_data {
        Ok(current_data) => {
            let new_info = read_version_info(new_data)?;
            let current_info = read_version_info(&current_data);

            match (
                new_info.get("version").and_then(|v| v.as_str()),
                current_info,
            ) {
                (Some(new_version), Ok(current_info)) => {
                    let current_version = current_info
                        .get("version")
                        .and_then(|v| v.as_str())
                        .unwrap_or("");

                    if current_version.is_empty()
                        || Version::parse(new_version)? > Version::parse(current_version)?
                    {
                        write_file_contents(filename, new_data)?;
                        info!("{} → {}", current_version, new_version);
                        Ok(true)
                    } else {
                        Ok(false)
                    }
                }
                // 如果读取当前版本信息时出错，则更新文件
                (Some(_), Err(_)) => {
                    write_file_contents(filename, new_data)?;
                    Ok(true)
                }
                (None, _) => {
                    // 如果读取新版本信息时出错，则不更新文件
                    Ok(false)
                }
            }
        }
        Err(_) => {
            // 如果读取当前文件时出错，则创建新文件
            write_file_contents(filename, new_data)?;
            Ok(true)
        }
    }
}

```



# main.rs

```rust
// 防止在 Windows 发布版本中出现额外的控制台窗口，请勿删除！！
#![cfg_attr(not(debug_assertions), windows_subsystem = "windows")]

// 导入 core 模块
mod core;
use core::{cmd, setup};

// 主函数，应用程序的入口点
fn main() {
    // 创建 Tauri 应用构建器
    tauri::Builder::default()
        // 初始化操作系统插件
        .plugin(tauri_plugin_os::init())
        // 初始化 Shell 插件
        .plugin(tauri_plugin_shell::init())
        // 初始化对话框插件
        .plugin(tauri_plugin_dialog::init())
        // 设置调用处理程序，处理前端调用的命令
        .invoke_handler(tauri::generate_handler![
            cmd::view_reload,         // 重新加载视图命令
            cmd::view_url,            // 视图 URL 命令
            cmd::view_go_forward,     // 视图前进命令
            cmd::view_go_back,        // 视图后退命令
            cmd::set_view_ask,        // 设置视图提问命令
            cmd::get_app_conf,        // 获取应用配置命令
            cmd::window_pin,          // 窗口置顶命令
            cmd::ask_sync,            // 提问同步命令
            cmd::ask_send,            // 提问发送命令
            cmd::set_theme,           // 设置主题命令
        ])
        // 设置初始化函数
        .setup(setup::init)
        // 运行 Tauri 应用
        .run(tauri::generate_context!())
        // 如果运行过程中出错，打印错误信息
        .expect("error while running lencx/ChatGPT application");
}

```



# build.rs

```rust
// src-tauri/build.rs

// 这个函数是构建脚本的入口点
fn main() {
    // 设置环境变量 MACOSX_DEPLOYMENT_TARGET 的值为 10.13
    // 这指定了最低支持的 macOS 版本
    println!("cargo:rustc-env=MACOSX_DEPLOYMENT_TARGET=10.13");

    // 调用 tauri_build 库的 build 函数来执行构建
    tauri_build::build()
}

```

