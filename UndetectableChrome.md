# Undetectable Chrome Driver

GitHub Repository: [Link](https://github.com/UltrafunkAmsterdam/undetected-chromedriver)

## Introduction

The `undetected-chromedriver` repository is a custom implementation of Selenium's Chrome WebDriver designed to bypass detection mechanisms employed by various anti-bot services, such as Distil Networks, Imperva, DataDome, and CloudFlare's IUAM. citeturn0search0

**How It Works:**

`undetected-chromedriver` operates by modifying the standard ChromeDriver in ways that prevent websites from recognizing automation scripts. It automatically downloads and patches the ChromeDriver binary to avoid triggering anti-bot services. This allows developers to run Selenium-based automation tasks without being blocked or challenged by protective measures on websites. citeturn0search0

**Differences from Pure Selenium Framework in Python:**

1. **Detection Avoidance:** Standard Selenium WebDriver instances can be easily detected by sophisticated anti-bot systems, leading to blocks or CAPTCHAs. `undetected-chromedriver` specifically addresses these detection methods, enabling scripts to run more stealthily. citeturn0search0

2. **Automatic Patching:** While using Selenium typically requires manual management of the ChromeDriver binary, `undetected-chromedriver` automates the process by downloading and patching the driver to enhance its undetectability. citeturn0search0

3. **Additional Features:** `undetected-chromedriver` offers extra functionalities, such as support for Brave Browser and other Chromium-based browsers, provided some configurations are adjusted. It also includes methods like `click_safe()` and `children()` to improve interaction with web elements.

In summary, `undetected-chromedriver` extends the capabilities of the standard Selenium framework by focusing on evading detection from anti-bot services, automating driver management, and providing additional tools for web automation tasks. 

> [!IMPORTANT]
> This framework doesn't change your IP address, so, when you running it from your home or even ISP servers, changes to bypass bot detection systems are low.

### How it's actually working?

Repository structure:

```
.
├── cdp.py
├── devtool.py
├── dprocess.py
├── __init__.py
├── options.py
├── patcher.py
├── reactor.py
└── webelement.py
```

#### __init__ .py

When you import Chrome from undetectable_chromedriver package, You're actually importing **Chrome** class from this file.

```py
class Chrome(selenium.webdriver.chrome.webdriver.WebDriver):

    _instances = set()
    session_id = None
    debug = False

    def __init__(
        self,
        options=None,
        user_data_dir=None,
        driver_executable_path=None,
        browser_executable_path=None,
        port=0,
        enable_cdp_events=False,
        # service_args=None,
        # service_creationflags=None,
        desired_capabilities=None,
        advanced_elements=False,
        # service_log_path=None,
        keep_alive=True,
        log_level=0,
        headless=False,
        version_main=None,
        patcher_force_close=False,
        suppress_welcome=True,
        use_subprocess=True,
        debug=False,
        no_sandbox=True,
        user_multi_procs: bool = False,
        **kw,
    ):
        
        finalize(self, self._ensure_close, self)
        self.debug = debug
        self.patcher = Patcher(
            executable_path=driver_executable_path,
            force=patcher_force_close,
            version_main=version_main,
            user_multi_procs=user_multi_procs,
        )
        # self.patcher.auto(user_multiprocess = user_multi_num_procs)
        self.patcher.auto()

        # self.patcher = patcher
        if not options:
            options = ChromeOptions()

        try:
            if hasattr(options, "_session") and options._session is not None:
                #  prevent reuse of options,
                #  as it just appends arguments, not replace them
                #  you'll get conflicts starting chrome
                raise RuntimeError("you cannot reuse the ChromeOptions object")
        except AttributeError:
            pass

        options._session = self

        if not options.debugger_address:
            debug_port = (
                port
                if port != 0
                else selenium.webdriver.common.service.utils.free_port()
            )
            debug_host = "127.0.0.1"
            options.debugger_address = "%s:%d" % (debug_host, debug_port)
        else:
            debug_host, debug_port = options.debugger_address.split(":")
            debug_port = int(debug_port)

        if enable_cdp_events:
            options.set_capability(
                "goog:loggingPrefs", {"performance": "ALL", "browser": "ALL"}
            )

        options.add_argument("--remote-debugging-host=%s" % debug_host)
        options.add_argument("--remote-debugging-port=%s" % debug_port)

        if user_data_dir:
            options.add_argument("--user-data-dir=%s" % user_data_dir)

        language, keep_user_data_dir = None, bool(user_data_dir)

        # see if a custom user profile is specified in options
        for arg in options.arguments:

            if any([_ in arg for _ in ("--headless", "headless")]):
                options.arguments.remove(arg)
                options.headless = True

            if "lang" in arg:
                m = re.search("(?:--)?lang(?:[ =])?(.*)", arg)
                try:
                    language = m[1]
                except IndexError:
                    logger.debug("will set the language to en-US,en;q=0.9")
                    language = "en-US,en;q=0.9"

            if "user-data-dir" in arg:
                m = re.search("(?:--)?user-data-dir(?:[ =])?(.*)", arg)
                try:
                    user_data_dir = m[1]
                    logger.debug(
                        "user-data-dir found in user argument %s => %s" % (arg, m[1])
                    )
                    keep_user_data_dir = True

                except IndexError:
                    logger.debug(
                        "no user data dir could be extracted from supplied argument %s "
                        % arg
                    )

        if not user_data_dir:
            # backward compatiblity
            # check if an old uc.ChromeOptions is used, and extract the user data dir

            if hasattr(options, "user_data_dir") and getattr(
                options, "user_data_dir", None
            ):
                import warnings

                warnings.warn(
                    "using ChromeOptions.user_data_dir might stop working in future versions."
                    "use uc.Chrome(user_data_dir='/xyz/some/data') in case you need existing profile folder"
                )
                options.add_argument("--user-data-dir=%s" % options.user_data_dir)
                keep_user_data_dir = True
                logger.debug(
                    "user_data_dir property found in options object: %s" % user_data_dir
                )

            else:
                user_data_dir = os.path.normpath(tempfile.mkdtemp())
                keep_user_data_dir = False
                arg = "--user-data-dir=%s" % user_data_dir
                options.add_argument(arg)
                logger.debug(
                    "created a temporary folder in which the user-data (profile) will be stored during this\n"
                    "session, and added it to chrome startup arguments: %s" % arg
                )

        if not language:
            try:
                import locale

                language = locale.getdefaultlocale()[0].replace("_", "-")
            except Exception:
                pass
            if not language:
                language = "en-US"

        options.add_argument("--lang=%s" % language)

        if not options.binary_location:
            options.binary_location = (
                browser_executable_path or find_chrome_executable()
            )

        if not options.binary_location or not \
                pathlib.Path(options.binary_location).exists():
                raise FileNotFoundError(
                    "\n---------------------\n"
                    "Could not determine browser executable."
                    "\n---------------------\n"
                    "Make sure your browser is installed in the default location (path).\n"
                    "If you are sure about the browser executable, you can specify it using\n"
                    "the `browser_executable_path='{}` parameter.\n\n"
                    .format("/path/to/browser/executable" if IS_POSIX else "c:/path/to/your/browser.exe")
                )

        self._delay = 3

        self.user_data_dir = user_data_dir
        self.keep_user_data_dir = keep_user_data_dir

        if suppress_welcome:
            options.arguments.extend(["--no-default-browser-check", "--no-first-run"])
        if no_sandbox:
            options.arguments.extend(["--no-sandbox", "--test-type"])

        if headless or getattr(options, 'headless', None):
            #workaround until a better checking is found
            try:
                if self.patcher.version_main < 108:
                    options.add_argument("--headless=chrome")
                elif self.patcher.version_main >= 108:
                    options.add_argument("--headless=new")
            except:
                logger.warning("could not detect version_main."
                               "therefore, we are assuming it is chrome 108 or higher")
                options.add_argument("--headless=new")

        options.add_argument("--window-size=1920,1080")
        options.add_argument("--start-maximized")
        options.add_argument("--no-sandbox")
        # fixes "could not connect to chrome" error when running
        # on linux using privileged user like root (which i don't recommend)

        options.add_argument(
            "--log-level=%d" % log_level
            or divmod(logging.getLogger().getEffectiveLevel(), 10)[0]
        )

        if hasattr(options, "handle_prefs"):
            options.handle_prefs(user_data_dir)

        # fix exit_type flag to prevent tab-restore nag
        try:
            with open(
                os.path.join(user_data_dir, "Default/Preferences"),
                encoding="latin1",
                mode="r+",
            ) as fs:
                config = json.load(fs)
                if config["profile"]["exit_type"] is not None:
                    # fixing the restore-tabs-nag
                    config["profile"]["exit_type"] = None
                fs.seek(0, 0)
                json.dump(config, fs)
                fs.truncate()  # the file might be shorter
                logger.debug("fixed exit_type flag")
        except Exception as e:
            logger.debug("did not find a bad exit_type flag ")

        self.options = options

        if not desired_capabilities:
            desired_capabilities = options.to_capabilities()

        if not use_subprocess:
            self.browser_pid = start_detached(
                options.binary_location, *options.arguments
            )
        else:
            browser = subprocess.Popen(
                [options.binary_location, *options.arguments],
                stdin=subprocess.PIPE,
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE,
                close_fds=IS_POSIX,
            )
            self.browser_pid = browser.pid


        service = selenium.webdriver.chromium.service.ChromiumService(
            self.patcher.executable_path
        )

        super(Chrome, self).__init__(
            service=service,
            options=options,
            keep_alive=keep_alive,
        )

        self.reactor = None

        if enable_cdp_events:
            if logging.getLogger().getEffectiveLevel() == logging.DEBUG:
                logging.getLogger(
                    "selenium.webdriver.remote.remote_connection"
                ).setLevel(20)
            reactor = Reactor(self)
            reactor.start()
            self.reactor = reactor

        if advanced_elements:
            self._web_element_cls = UCWebElement
        else:
            self._web_element_cls = WebElement

        if headless or getattr(options, 'headless', None):
            self._configure_headless()

    def _configure_headless(self):
        orig_get = self.get
        logger.info("setting properties for headless")

        def get_wrapped(*args, **kwargs):
            if self.execute_script("return navigator.webdriver"):
                logger.info("patch navigator.webdriver")
                self.execute_cdp_cmd(
                    "Page.addScriptToEvaluateOnNewDocument",
                    {
                        "source": """

                           Object.defineProperty(window, "navigator", {
                                Object.defineProperty(window, "navigator", {
                                  value: new Proxy(navigator, {
                                    has: (target, key) => (key === "webdriver" ? false : key in target),
                                    get: (target, key) =>
                                      key === "webdriver"
                                        ? false
                                        : typeof target[key] === "function"
                                        ? target[key].bind(target)
                                        : target[key],
                                  }),
                                });
                    """
                    },
                )

                logger.info("patch user-agent string")
                self.execute_cdp_cmd(
                    "Network.setUserAgentOverride",
                    {
                        "userAgent": self.execute_script(
                            "return navigator.userAgent"
                        ).replace("Headless", "")
                    },
                )
                self.execute_cdp_cmd(
                    "Page.addScriptToEvaluateOnNewDocument",
                    {
                        "source": """
                            Object.defineProperty(navigator, 'maxTouchPoints', {get: () => 1});
                            Object.defineProperty(navigator.connection, 'rtt', {get: () => 100});

                            // https://github.com/microlinkhq/browserless/blob/master/packages/goto/src/evasions/chrome-runtime.js
                            window.chrome = {
                                app: {
                                    isInstalled: false,
                                    InstallState: {
                                        DISABLED: 'disabled',
                                        INSTALLED: 'installed',
                                        NOT_INSTALLED: 'not_installed'
                                    },
                                    RunningState: {
                                        CANNOT_RUN: 'cannot_run',
                                        READY_TO_RUN: 'ready_to_run',
                                        RUNNING: 'running'
                                    }
                                },
                                runtime: {
                                    OnInstalledReason: {
                                        CHROME_UPDATE: 'chrome_update',
                                        INSTALL: 'install',
                                        SHARED_MODULE_UPDATE: 'shared_module_update',
                                        UPDATE: 'update'
                                    },
                                    OnRestartRequiredReason: {
                                        APP_UPDATE: 'app_update',
                                        OS_UPDATE: 'os_update',
                                        PERIODIC: 'periodic'
                                    },
                                    PlatformArch: {
                                        ARM: 'arm',
                                        ARM64: 'arm64',
                                        MIPS: 'mips',
                                        MIPS64: 'mips64',
                                        X86_32: 'x86-32',
                                        X86_64: 'x86-64'
                                    },
                                    PlatformNaclArch: {
                                        ARM: 'arm',
                                        MIPS: 'mips',
                                        MIPS64: 'mips64',
                                        X86_32: 'x86-32',
                                        X86_64: 'x86-64'
                                    },
                                    PlatformOs: {
                                        ANDROID: 'android',
                                        CROS: 'cros',
                                        LINUX: 'linux',
                                        MAC: 'mac',
                                        OPENBSD: 'openbsd',
                                        WIN: 'win'
                                    },
                                    RequestUpdateCheckStatus: {
                                        NO_UPDATE: 'no_update',
                                        THROTTLED: 'throttled',
                                        UPDATE_AVAILABLE: 'update_available'
                                    }
                                }
                            }

                            // https://github.com/microlinkhq/browserless/blob/master/packages/goto/src/evasions/navigator-permissions.js
                            if (!window.Notification) {
                                window.Notification = {
                                    permission: 'denied'
                                }
                            }

                            const originalQuery = window.navigator.permissions.query
                            window.navigator.permissions.__proto__.query = parameters =>
                                parameters.name === 'notifications'
                                    ? Promise.resolve({ state: window.Notification.permission })
                                    : originalQuery(parameters)

                            const oldCall = Function.prototype.call
                            function call() {
                                return oldCall.apply(this, arguments)
                            }
                            Function.prototype.call = call

                            const nativeToStringFunctionString = Error.toString().replace(/Error/g, 'toString')
                            const oldToString = Function.prototype.toString

                            function functionToString() {
                                if (this === window.navigator.permissions.query) {
                                    return 'function query() { [native code] }'
                                }
                                if (this === functionToString) {
                                    return nativeToStringFunctionString
                                }
                                return oldCall.call(oldToString, this)
                            }
                            // eslint-disable-next-line
                            Function.prototype.toString = functionToString
                            """
                    },
                )
            return orig_get(*args, **kwargs)

        self.get = get_wrapped

    def get(self, url):
        return super().get(url)

    def add_cdp_listener(self, event_name, callback):
        if (
            self.reactor
            and self.reactor is not None
            and isinstance(self.reactor, Reactor)
        ):
            self.reactor.add_event_handler(event_name, callback)
            return self.reactor.handlers
        return False

    def clear_cdp_listeners(self):
        if self.reactor and isinstance(self.reactor, Reactor):
            self.reactor.handlers.clear()

    def window_new(self):
        self.execute(
            selenium.webdriver.remote.command.Command.NEW_WINDOW, {"type": "window"}
        )

    def tab_new(self, url: str):
        """
        this opens a url in a new tab.
        apparently, that passes all tests directly!

        Parameters
        ----------
        url

        Returns
        -------

        """
        if not hasattr(self, "cdp"):
            from .cdp import CDP

            cdp = CDP(self.options)
            cdp.tab_new(url)

    def reconnect(self, timeout=0.1):
        try:
            self.service.stop()
        except Exception as e:
            logger.debug(e)
        time.sleep(timeout)
        try:
            self.service.start()
        except Exception as e:
            logger.debug(e)

        try:
            self.start_session()
        except Exception as e:
            logger.debug(e)

    def start_session(self, capabilities=None, browser_profile=None):
        if not capabilities:
            capabilities = self.options.to_capabilities()
        super(selenium.webdriver.chrome.webdriver.WebDriver, self).start_session(
            capabilities
        )

    def find_elements_recursive(self, by, value):
        """
        find elements in all frames
        this is a generator function, which is needed
            since if it would return a list of elements, they
            will be stale on arrival.
        using generator, when the element is returned we are in the correct frame
        to use it directly
        Args:
            by: By
            value: str
        Returns: Generator[webelement.WebElement]
        """
        def search_frame(f=None):
            if not f:
                # ensure we are on main content frame
                self.switch_to.default_content()
            else:
                self.switch_to.frame(f)
            for elem in self.find_elements(by, value):
                yield elem
            # switch back to main content, otherwise we will get StaleElementReferenceException
            self.switch_to.default_content()

        # search root frame
        for elem in search_frame():
            yield elem
        # get iframes
        frames = self.find_elements('css selector', 'iframe')

        # search per frame
        for f in frames:
            for elem in search_frame(f):
                yield elem

    def quit(self):
        try:
            self.service.process.kill()
            logger.debug("webdriver process ended")
        except (AttributeError, RuntimeError, OSError):
            pass
        try:
            self.reactor.event.set()
            logger.debug("shutting down reactor")
        except AttributeError:
            pass
        try:
            os.kill(self.browser_pid, 15)
            logger.debug("gracefully closed browser")
        except Exception as e:  # noqa
            pass
        if (
            hasattr(self, "keep_user_data_dir")
            and hasattr(self, "user_data_dir")
            and not self.keep_user_data_dir
        ):
            for _ in range(5):
                try:
                    shutil.rmtree(self.user_data_dir, ignore_errors=False)
                except FileNotFoundError:
                    pass
                except (RuntimeError, OSError, PermissionError) as e:
                    logger.debug(
                        "When removing the temp profile, a %s occured: %s\nretrying..."
                        % (e.__class__.__name__, e)
                    )
                else:
                    logger.debug("successfully removed %s" % self.user_data_dir)
                    break
                time.sleep(0.1)

        # dereference patcher, so patcher can start cleaning up as well.
        # this must come last, otherwise it will throw 'in use' errors
        self.patcher = None

    def __getattribute__(self, item):
        if not super().__getattribute__("debug"):
            return super().__getattribute__(item)
        else:
            import inspect

            original = super().__getattribute__(item)
            if inspect.ismethod(original) and not inspect.isclass(original):

                def newfunc(*args, **kwargs):
                    logger.debug(
                        "calling %s with args %s and kwargs %s\n"
                        % (original.__qualname__, args, kwargs)
                    )
                    return original(*args, **kwargs)

                return newfunc
            return original

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.service.stop()
        time.sleep(self._delay)
        self.service.start()
        self.start_session()

    def __hash__(self):
        return hash(self.options.debugger_address)

    def __dir__(self):
        return object.__dir__(self)

    def __del__(self):
        try:
            self.service.process.kill()
        except:  # noqa
            pass
        self.quit()

    @classmethod
    def _ensure_close(cls, self):
        # needs to be a classmethod so finalize can find the reference
        logger.info("ensuring close")
        if (
            hasattr(self, "service")
            and hasattr(self.service, "process")
            and hasattr(self.service.process, "kill")
        ):
            self.service.process.kill()

```

