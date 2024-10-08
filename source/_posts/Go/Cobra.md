---
title: Cobra
date: 2024-08-24 16:55:04
tags:
categories:
- Go
---

- [初始化项目](#初始化项目)
- [cobra结构体](#cobra结构体)
- [init](#init)
- [flag](#flag)
- [xxxCmd结构体方法执行顺序](#xxxcmd结构体方法执行顺序)
- [隐藏某个cobra命令的提示信息](#隐藏某个cobra命令的提示信息)


参考链接：

	https://github.com/spf13/cobra
	https://github.com/spf13/cobra/blob/main/user_guide.md
	https://github.com/spf13/cobra-cli/blob/main/README.md
	https://blog.kelu.org/tech/2021/04/10/kubernetes-cobra.html



### 初始化项目

先go mod init 包名来初始化一个go.mod(参考本目录文章《goModInit》)

	go mod init github.com/senranja/PPP

cd进入PPP目录后，使用cobra命令进行初始化

	cobra-cli init  --license apache  --author "Steve Francia spf@spf13.com"  初始化cobra项目
	cobra-cli add serve  添加子命令
	cobra-cli add create -p 'configCmd' 指定父级命令创建子命令

### cobra结构体

其中app/cmd/root.go中，主要是围绕结构体 xxxCmd 来设置Use、Args、Long、Short、Run系列、等内容

//可以自定义任意Execute，反正得执行了 rootCmd.Execute() 这个函数

	func Execute() error {
		return rootCmd.Execute()
	}


### init

//初始化的一个函数，在这里绑定各种flag。其中有P的时有缩写的，没P的是没有缩写的，没P的通常--radact就可以表示没有什么值
	
	func init() {
		cobra.OnInitialize(initConfig) //用来初始化日志级别相关的东西
	}

我们知道 init 函数是 Golang 中初始化包的时候第一个调用的函数。在 cmd/root.go 中我们可以看到 init 函数中调用了 cobra.OnInitialize(initConfig)，也就是每当执行或者调用命令的时候，它都会先执行 init 函数中的所有函数，然后再执行 execute 方法。该初始化可用于加载配置文件或用于构造函数等等，这完全依赖于我们应用的实际情况。

在初始化函数里面 cobra.OnInitialize(initConfig) 调用了 initConfig 这个函数，所有，当 rootCmd 的执行方法 RUN: func 运行的时候，rootCmd 根命令就会首先运行 initConfig 函数，当所有的初始化函数执行完成后，才会执行 rootCmd 的 RUN: func 执行函数。

	func initConfig() {
		if cfgFile != "" {
			// Use config file from the flag.
			viper.SetConfigFile(cfgFile)
		} else {
			// Find home directory.
			home, err := os.UserHomeDir()
			cobra.CheckErr(err)

			// Search config in home directory with name ".cobra" (without extension).
			viper.AddConfigPath(home)
			viper.SetConfigType("yaml")
			viper.SetConfigName(".cobra")
		}

		viper.AutomaticEnv()

		if err := viper.ReadInConfig(); err == nil {
			fmt.Println("Using config file:", viper.ConfigFileUsed())
		}
	}


### flag

	rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.cobra.yaml)")
	rootCmd.PersistentFlags().StringP("author", "a", "YOUR NAME", "author name for copyright attribution")
	rootCmd.PersistentFlags().StringVarP(&userLicense, "license", "l", "", "name of license for the project")
	rootCmd.PersistentFlags().Bool("viper", true, "use Viper for configuration")
	viper.BindPFlag("author", rootCmd.PersistentFlags().Lookup("author"))
	viper.BindPFlag("useViper", rootCmd.PersistentFlags().Lookup("viper"))
	viper.SetDefault("author", "NAME HERE <EMAIL ADDRESS>")
	viper.SetDefault("license", "apache")

	rootCmd.AddCommand(addCmd)
	rootCmd.AddCommand(initCmd)

### xxxCmd结构体方法执行顺序

	PersistentPreRun
	PreRun
	Run
	PostRun
	PersistentPostRun

### 隐藏某个cobra命令的提示信息

	rootCmd.PersistentFlags().StringP("config", "", "", "")
	可以使用 MarkHidden() 函数来表示某个信息的隐藏
	rootCmd.PersistentFlags().MarkHidden("config")








