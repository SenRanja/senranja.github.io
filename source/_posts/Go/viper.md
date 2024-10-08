---
title: viper
date: 2024-08-24 15:09:17
tags:
categories:
- Go
---

github.com/spf13/viper: 读取配置文件

- [优先级](#优先级)
- [直接设置环境](#直接设置环境)
- [读取配置文件](#读取配置文件)
    - [1. viper.ReadConfig: # 读取raw的配置文件本身](#1-viperreadconfig--读取raw的配置文件本身)
    - [2. viper.ReadInConfig() # viper.ReadInConfig()读取某个配置文件](#2-viperreadinconfig--viperreadinconfig读取某个配置文件)
- [写配置文件](#写配置文件)
- [Run time时盯配置文件是否有异动热加载配置。因此不需要重启服务器，就能让配置生效。](#run-time时盯配置文件是否有异动热加载配置因此不需要重启服务器就能让配置生效)
- [gitleaks案例:读取\[\[Rule\]\]这种列表的toml数据](#gitleaks案例读取rule这种列表的toml数据)
- [设置别名](#设置别名)
- [从io.Reader读取配置，例：读取bindata打包文件到viper](#从ioreader读取配置例读取bindata打包文件到viper)
- [绑定环境变量](#绑定环境变量)
- [前缀的设置](#前缀的设置)
- [Unmarshal到结构体中](#unmarshal到结构体中)
- [viper和cobra结合](#viper和cobra结合)
- [viper和pflag结合](#viper和pflag结合)
- [gitleaks案例：同viper要求读取2个配置文件，后者覆盖前者](#gitleaks案例同viper要求读取2个配置文件后者覆盖前者)
- [将viper中的toml读取到结构体](#将viper中的toml读取到结构体)


参考链接：
    https://github.com/spf13/viper
    https://juejin.cn/post/6844904051369312264


### 优先级

    explicit call to Set
    flag
    env
    config
    key/value store
    default

### 直接设置环境

    viper.SetDefault("ContentDir", "content")
    viper.SetDefault("LayoutDir", "layouts")
    viper.SetDefault("Taxonomies", map[string]string{"tag": "tags", "category": "categories"})

### 读取配置文件

支持配置文件格式：JSON, TOML, YAML, HCL, INI, envfile and Java Properties files

注意：viper的配置的键值是大小写不敏感的，对环境变量是大小写敏感的。一个viper实例支持多个路径环境（但是至少得提供一个），但是只支持解析1个配置文件

另外注意，在写gitleaks打包项目时注意到，viper在进行配置时可以配置路径、原生的配置文件、后缀名等等，但是读取的行为基本上时这两个函数：
Eg:
##### 1. viper.ReadConfig: # 读取raw的配置文件本身
   
    bindata_default_toml, _ := bindata.Asset("gitleaks-n-all-kill.toml")
    viper.SetConfigType("toml")
    if err := viper.ReadConfig(bytes.NewBuffer(bindata_default_toml)); err != nil {
        log.Fatal().Msgf("unable to load gitleaks config, err: %s", err)
    }

##### 2. viper.ReadInConfig() # viper.ReadInConfig()读取某个配置文件
   
    viper.SetConfigFile(cfgPath)  		
    fmt.Print("已加载config参数")
    if err := viper.ReadInConfig(); err != nil {
        log.Fatal().Msgf("unable to load gitleaks config, err: %s", err)
    }

    viper.SetConfigName("config") // name of config file (without extension)
    viper.SetConfigType("yaml") // REQUIRED if the config file does not have the extension in the name
    viper.AddConfigPath("/etc/appname/")   // path to look for the config file in
    viper.AddConfigPath("$HOME/.appname")  // call multiple times to add many search paths
    viper.AddConfigPath(".")               // optionally look for config in the working directory
    err := viper.ReadInConfig() // Find and read the config file
    if err != nil { // Handle errors reading the config file
        panic(fmt.Errorf("fatal error config file: %w", err))
    }

    if err := viper.ReadInConfig(); err != nil {
        if _, ok := err.(viper.ConfigFileNotFoundError); ok {
            // Config file not found; ignore error if desired
        } else {
            // Config file was found but another error was produced
        }
    }
    // Config file found and successfully parsed

另一个案例

    package main 
    import ( "fmt" "log" "github.com/spf13/viper" ) 
    func main() 
    { 	viper.SetConfigName("config") 
    viper.SetConfigType("toml") 
    viper.AddConfigPath(".") 

    //viper.ReadInConfig() 之前，可以先设置个默认值，配置文件此时还没解析，解析出来他自动覆盖这个默认值
    viper.SetDefault("redis.port", 6381)

    err := viper.ReadInConfig() 
    if err != nil { log.Fatal("read config failed: %v", err) } 
    fmt.Println(viper.Get("app_name")) 
    fmt.Println(viper.Get("log_level")) 
    fmt.Println("mysql ip: ", viper.Get("mysql.ip")) 
    fmt.Println("mysql port: ", viper.Get("mysql.port")) 
    fmt.Println("mysql user: ", viper.Get("mysql.user")) 
    fmt.Println("mysql password: ", viper.Get("mysql.password")) 
    fmt.Println("mysql database: ", viper.Get("mysql.database")) 
    fmt.Println("redis ip: ", viper.Get("redis.ip")) 
    fmt.Println("redis port: ", viper.Get("redis.port")) 
    }

### 写配置文件

    viper.WriteConfig() // writes current config to predefined path set by 'viper.AddConfigPath()' and 'viper.SetConfigName'
    viper.SafeWriteConfig()
    viper.WriteConfigAs("/path/to/my/.config")
    viper.SafeWriteConfigAs("/path/to/my/.config") // will error since it has already been written
    viper.SafeWriteConfigAs("/path/to/my/.other_config")

### Run time时盯配置文件是否有异动热加载配置。因此不需要重启服务器，就能让配置生效。

    viper.OnConfigChange(func(e fsnotify.Event) {
        fmt.Println("Config file changed:", e.Name)
    })
    viper.WatchConfig()

如：
    package main

    import (
    "fmt"
    "log"
    "time"

    "github.com/spf13/viper"
    )

    func main() {
    viper.SetConfigName("config")
    viper.SetConfigType("toml")
    viper.AddConfigPath(".")
    err := viper.ReadInConfig()
    if err != nil {
        log.Fatal("read config failed: %v", err)
    }

    viper.WatchConfig()

    fmt.Println("redis port before sleep: ", viper.Get("redis.port"))
    time.Sleep(time.Second * 10)
    fmt.Println("redis port after sleep: ", viper.Get("redis.port"))
    }

只需要调用`viper.WatchConfig`，viper 会自动监听配置修改。如果有修改，重新加载的配置。
上面程序中，我们先打印redis.port的值，然后Sleep 10s。在这期间修改配置中redis.port的值，Sleep结束后再次打印。

另外，还可以为配置修改增加一个回调：

    viper.OnConfigChange(func(e fsnotify.Event) {
    fmt.Printf("Config file:%s Op:%s\n", e.Name, e.Op)
    })

这样文件修改时会执行这个回调。

viper 使用`fsnotify`这个库来实现监听文件修改的功能。

### gitleaks案例:读取[[Rule]]这种列表的toml数据

如这里结构体写的 Rules 是一个 []

    type ViperConfig struct {
        Description string
        Extend      Extend
        Rules       []struct {
            ID          string
            Description string
            Entropy     float64
            SecretGroup int
            Regex       string
            Keywords    []string
            Path        string
            Tags        []string

            Allowlist struct {
                Regexes   []string
                Paths     []string
                Commits   []string
                StopWords []string
            }
        }
        Allowlist struct {
            Regexes   []string
            Paths     []string
            Commits   []string
            StopWords []string
        }
    }

在toml中，其配置内容如下，是多个`[[Rules]]`这种写法

    # [ GitLeaks原本的规则 ]

    [[rules]]
    description = "应用凭证--Adafruit API Key"
    id = "adafruit-api-key"
    regex = '''(?i)(?:adafruit)(?:[0-9a-z\-_\t .]{0,20})(?:[\s|']|[\s|"]){0,3}(?:=|>|:=|\|\|:|<=|=>|:)(?:'|\"|\s|=|\x60){0,5}([a-z0-9_-]{32})(?:['|\"|\n|\r|\s|\x60|;]|$)'''
    secretGroup = 1
    keywords = [
        "adafruit",
    ]

    [[rules]]
    description = "应用凭证--Adobe Client ID (OAuth Web)"
    id = "adobe-client-id"
    regex = '''(?i)(?:adobe)(?:[0-9a-z\-_\t .]{0,20})(?:[\s|']|[\s|"]){0,3}(?:=|>|:=|\|\|:|<=|=>|:)(?:'|\"|\s|=|\x60){0,5}([a-f0-9]{32})(?:['|\"|\n|\r|\s|\x60|;]|$)'''
    secretGroup = 1
    keywords = [
        "adobe",
    ]

### 设置别名

    viper.RegisterAlias("loud", "Verbose")
    viper.Set("verbose", true) // same result as next line
    viper.Set("loud", true)   // same result as prior line
    viper.GetBool("loud") // true
    viper.GetBool("verbose") // true


### 从io.Reader读取配置，例：读取bindata打包文件到viper

    viper.SetConfigType("yaml") // or viper.SetConfigType("YAML")

    // any approach to require this configuration into your program.
    var yamlExample = []byte(`
    Hacker: true
    name: steve
    hobbies:
    - skateboarding
    - snowboarding
    - go
    clothing:
    jacket: leather
    trousers: denim
    age: 35
    eyes : brown
    beard: true
    `)

    viper.ReadConfig(bytes.NewBuffer(yamlExample))

    viper.Get("name") // this would be "steve"

### 绑定环境变量

    func init() { 
    // 绑定环境变量 
    viper.BindEnv("redis.port") 
    viper.BindEnv("go.path", "GOPATH") 
    } 

    func main() { 
    // 省略部分代码 
    fmt.Println("go path: ", viper.Get("go.path")) 
    }

调用BindEnv方法，如果只传入一个参数，则这个参数既表示键名，又表示环境变量名。 如果传入两个参数，则第一个参数表示键名，第二个参数表示环境变量名。

还可以通过viper.SetEnvPrefix方法设置环境变量前缀，这样一来，通过AutomaticEnv和一个参数的BindEnv绑定的环境变量， 在使用Get的时候，viper 会自动加上这个前缀再从环境变量中查找。

如果对应的环境变量不存在，viper 会自动将键名全部转为大写再查找一次。所以，使用键名gopath也能读取环境变量GOPATH的值。

### 前缀的设置

    SetEnvPrefix("spf") // will be uppercased automatically
    BindEnv("id")

    os.Setenv("SPF_ID", "13") // typically done outside of the app

    id := Get("id") // 13

### Unmarshal到结构体中

    package main import ( "fmt" "log" "github.com/spf13/viper" ) 
    type Config struct 
    { AppName string 
    LogLevel string 
    MySQL MySQLConfig 
    Redis RedisConfig 
    } 
    type MySQLConfig struct { 
    IP string 
    Port int 
    User string 
    Password string 
    Database string 
    } 

    type RedisConfig struct { IP string Port int } 

    func main() { 
    viper.SetConfigName("config") 
    viper.SetConfigType("toml") 
    viper.AddConfigPath(".") 
    err := viper.ReadInConfig() 
    if err != nil { 
    log.Fatal("read config failed: %v", err) 
    } 
    var c Config 
    viper.Unmarshal(&c) 
    fmt.Println(c.MySQL) 
    }

### viper和cobra结合

    serverCmd.Flags().Int("port", 1138, "Port to run Application server on")
    viper.BindPFlag("port", serverCmd.Flags().Lookup("port"))

### viper和pflag结合

    pflag.Int("flagname", 1234, "help message for flagname")
    pflag.Parse()
    viper.BindPFlags(pflag.CommandLine)

    i := viper.GetInt("flagname") // retrieve values from viper instead of pflag

或者是：

    func init() { 
    pflag.Int("redis.port", 8381, "Redis port to connect") // 绑定命令行 
    viper.BindPFlags(pflag.CommandLine) 
    }

### gitleaks案例：同viper要求读取2个配置文件，后者覆盖前者

此次案例参考链接: https://treexie.gitbook.io/articles/viper

背景: 要求打包gitleaks，把toml的默认配置文件嵌套进去，然后该默认的配置文件我们可控，且支持用户对其中某个单条的配置规则的修改或者添加。

参考链接中示例如下:

    func initConfig() (err error) {
        configType := "yml"
        defaultPath := "./configs"
        v := viper.New()
        // 从default中读取默认的配置
        v.SetConfigName("default")
        v.AddConfigPath(defaultPath)
        v.SetConfigType(configType)
        err = v.ReadInConfig()
        if err != nil {
            return
        }

        configs := v.AllSettings()
        // 将default中的配置全部以默认配置写入
        for k, v := range configs {
            viper.SetDefault(k, v)
        } // 在这里使用到了viper.Set的方式设置值。 有Set()和SetDefault() 两个方法
        env := os.Getenv("GO_ENV")
        // 根据配置的env读取相应的配置信息
        if env != "" {
            viper.SetConfigName(env)
            viper.AddConfigPath(defaultPath)
            viper.SetConfigType(configType)
            err = viper.ReadInConfig()
    // 这里继续重复进行读取
            if err != nil {
                return
            }
        }
        return
    }



量来进行赋值传递

    // 处理[allowlist] 开始
            if user_custom_configs["allowlist"] != nil {
                var tmpConfigAllowlist map[string]interface{}
                tmpConfigAllowlist = make(map[string]interface{})
                for k, v := range configs["allowlist"].(map[string]interface{}) {
                    tmpConfigAllowlist[k] = v
                    //fmt.Println(k)
                    //fmt.Println(v)
                }
                user_custom_configs_allowlist := user_custom_configs["allowlist"].(map[string]interface{})
                for k, v := range user_custom_configs_allowlist {
                    if k == "paths" {
                        tmpConfigAllowlist[k] = v
                    }
                }
                configs["allowlist"] = tmpConfigAllowlist

            }

### 将viper中的toml读取到结构体

经过测试，标注的  Description string `toml:"description"`
此处最好不要写toml，而是写成 `mapstructure:"advice"`,会少一些读取不到的奇奇怪怪的错误

output.toml

    # gitlab version with risk
    [version]

    # admin setting
    [settings]
    [settings.password]
    check_rule="密码复杂度检测"
    check_rule_en="password_complex"
    description="密码复杂度检测"
    advice="密码复杂度检测"
    [settings.password.length]
    check_rule="最小长度"
    check_rule_en="password_least_length"
    description="最小长度"
    advice="要求数字"

写的结构体

    type Output struct {
        Settings struct {
            Password struct {
                CheckRule   string `mapstructure:"check_rule"`
                CheckRuleEn string `mapstructure:"check_rule_en"`
                Description string `mapstructure:"description"`
                Advice      string `mapstructure:"advice"`
                Length      struct {
                    CheckRule   string `mapstructure:"check_rule"`
                    Description string `mapstructure:"description"`
                    Advice      string `mapstructure:"advice"`
                } `toml:"length"`
                Num struct {
                    CheckRule   string `mapstructure:"check_rule"`
                    Description string `mapstructure:"description"`
                    Advice      string `mapstructure:"advice"`
                } `toml:"num"`
                Upper struct {
                    CheckRule   string `mapstructure:"check_rule"`
                    Description string `mapstructure:"description"`
                    Advice      string `mapstructure:"advice"`
                } `toml:"upper"`
                Lower struct {
                    CheckRule   string `mapstructure:"check_rule"`
                    Description string `toml:"description"`
                    Advice      string `toml:"advice"`
                } `toml:"lower"`
                Special struct {
                    CheckRule   string `toml:"check_rule"`
                    Description string `toml:"description"`
                    Advice      string `toml:"advice"`
                } `toml:"special"`
            } `toml:"password"`
        } `toml:"settings"`
    }

viper读取配置文件后读取到结构体中

    func (o *Output) GetDefault() {
        outputConfig := viper.New()
        bindataOutputDefaultToml, _ := bindata.Asset("output.toml")
        outputConfig.SetConfigType("toml")
        if err := outputConfig.ReadConfig(bytes.NewBuffer(bindataOutputDefaultToml)); err != nil {
            panic("unable to load output config")
            panic(err)
        }
        outputConfig.Unmarshal(o)
    }
