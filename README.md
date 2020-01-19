# Fastlane



[官方文档](https://docs.fastlane.tools)

## 安装

1. 安装最新版Xcode命令行工具
   
   ```shell
   xcode-select --install
   ```

2. 安装fastlane
   
   - 使用*Ruby*
   
   ```shell
   sudo gem install fastlane -NV
   ```
   
   - 或使用*homebrew*
   
   ```shell
   brew install faslane
   ```

## 使用

1. 进入工程目录，执行fastlane init
   
   ```shell
   fastlane init
   ```
   
   提示如下：4个选择，选个4
   
   ```ruby
   [08:56:33]: What would you like to use fastlane for?
   1. 📸 Automate screenshots
   2. 👩‍✈️ Automate beta distribution to TestFlight
   3. 🚀 Automate App Store distribution
   4. 🛠 Manual setup - manually setup your project to automate your tasks
   ?
   ```

2. 相信很多小伙伴卡在了 `$ bundle update` 这一步，没关系，在项目根目录找到 ***Gemfile***，编辑：
   
   ```ruby
   source "https://rubygems.org"
   替换为
   source "https://gems.ruby-china.com"
   ```
   
   回到终端，`ctrl + c`中断之后，输入：
   
   ```shell
   sudo bundle update
   ```

3. 导出ipa
   
   目录下已经多了fastlane文件夹，进入该文件夹，编辑 ***Fastfile***，执行的动作，都可以在[Fastfile Actions](https://docs.fastlane.tools/actions/)查询，导出ipa主要是用`gym(parameters...)`
   
   ```ruby
   # Fastfile
   default_platform(:ios)
   
   platform :ios do
      desc "Description of what the lane does"
      lane :custom_lane do
          # add actions here: https://docs.fastlane.tools/actions
          gym(scheme: "FastlaneDemo", 
              configuration: "Release",
              output_directory: "./fastlane/build",
              output_name: "FastlaneDemo",
              clean: true,
              export_method:"development")
    end
   end
   ```
   
   **[gym()参数表](https://docs.fastlane.tools/actions/gym/#parameters)：**
   
   | Key              | Description                         |
   |:---------------- | ----------------------------------- |
   | scheme           | 项目的scheme                           |
   | configuration    | 编辑配置，Release、Debug等等                |
   | clean            | 是否执行clean                           |
   | output_directory | 导出文件夹                               |
   | output_name      | 导出的ipa名称                            |
   | export_method    | development, add hoc, app-store ... |
   | ...              | ...                                 |

4. 打开终端，cd到工程根目录，输入`fastlane custom_lane`，这个custom_lane就是 fastfile里的定义的方法
   
   > 注意：fastlane打包时，会根据该项目当前的xcode证书配置来进行打包
   
   ```shell
   fastlane custom_lane
   ```

5. 关于 ***cocoapods*** 
   
   如果想在每次build之前，都进行 `pod install`，则需要在`gym()`前加入cocoapods操作
   
   ```ruby
   # Fastfile
   default_platform(:ios)
   platform :ios do
        desc "Description of what the lane does"
        lane :custom_lane do
            # add actions here: https://docs.fastlane.tools/actions
            # 执行pod install
            cocoapods
            # 执行打包，导出
            gym(scheme: "FastlaneDemo",
                configuration: "Release",
                output_directory: "./fastlane/build",
                output_name: "FastlaneDemo",
                clean: true,
                export_method:"development")
    end
   end
   ```
   
   执行`fastlane custom_lane`时，会报错:
   
   ```ruby
   Gem::Exception: can't find executable pod for gem cocoapods. cocoapods is not currently included in the bundle, perhaps you meant to add it to your Gemfile?
   ```
   
   编辑 ***Gemfile***，加入：
   
   ```ruby
   gem "cocoapods"
   ```
   
   重新执行，即可

简单的使用就到此结束了，如果小伙伴还需要上传到[蒲公英](http://www.pgyer.com/doc/view/fastlane)、[AppStore](https://docs.fastlane.tools/getting-started/ios/appstore-deployment/)、[TestFlight](https://docs.fastlane.tools/actions/testflight/)测试等等功能，请自行查阅官方文档

---

## 多Target和多Scheme的玩法

目的：根据Target和Scheme打包，导出对应ipa，并根据target + scheme + version + buildnumer命名ipa

#### Example:

1. 新增 ***.env.(自定义名字)*** 文件
   
   在**根目录/fastlane文件夹**下，根据不同的target生成不同的 ***.env*** 文件，Demo里包含了`.env.main`和`.env.another`
   
   > 注意：.env是隐藏文件，可以使用命令 ***command + shift + .*** 查看

2. 编辑 ***.env.(自定义名字)*** 文件，对应的项目结构如下表
   
   | Targets             | Bundle Identifier               | Scheme                             | .env文件       |
   | ------------------- | ------------------------------- | ---------------------------------- | ------------ |
   | FastlaneDemo        | com.wesoft.FastlaneDemo.main    | FastlaneDemo, FastlaneDemoQA       | .env.main    |
   | FastlaneDemoAnother | com.wesoft.FastlaneDemo.another | FastlaneAnother, FastlaneAnotherQA | .env.another |
   
   - `.env.main`填写对应的`APP_IDENTIFIER和SCHEME_NAME`
     
     ```ruby
     # .env.main
     APP_IDENTIFIER = "com.wesoft.FastlaneDemo.main"
     TARGET_NAME = "FastlaneDemo"
     SCHEME_NAME = "FaslaneDemo"
     SCHEME_NAME_QA = "FaslaneDemoQA"
     # more scheme ...
     ```
   
   - `.env.another`填写对应的`APP_IDENTIFIER和SCHEME_NAME`
     
     ```ruby
     # .env.another
     APP_IDENTIFIER = "com.wesoft.FastlaneDemo.main"
     TARGET_NAME = "FastlaneDemoAnother"
     SCHEME_NAME = "FaslaneDemoAnother"
     SCHEME_NAME_QA = "FaslaneDemoAnotherQA"
     # more scheme ...
     ```

3. [插件安装](https://docs.fastlane.tools/plugins/available-plugins/)
   
   因为在获取build number和version时，需要根据target对应的info.plist文件获取，fastlane原生的[get_build_number](https://docs.fastlane.tools/actions/get_build_number/)不支持，所以需要安装插件[fastlane-plugin-versioning](https://github.com/SiarheiFedartsou/fastlane-plugin-versioning)
   
   cd到项目根目录，打开终端，输入:
   
   ```
   fastlane add_plugin versioning
   ```
   
   安装好之后，就可以在 ***Gemfile*** 查看到了

4. 在项目的**info.plist**里将`Bundle version`的值改成`$(CURRENT_PROJECT_VERSION)`

5. 编辑 ***Fastfile***，步骤：
   
   5.1 新增`lane :deploy`方法
   
   5.2 实现get_build_number和version
   
   5.3 利用`ENV[变量名]`获取`.env.target`里的变量值
   
   5.4 新增批量执行的`lane :deploy_all`方法
   
   > 注意：`sh "fastlane deploy --env main"`里的`deploy`是上面定义的`lane :deploy`方法名，`--env main`是`-env`后面接`.env.main`文件的后缀
   
   ```ruby
   # Fastfile
   default_platform(:ios)
   
   platform :ios do
       desc "Deploy one target"
       lane :deploy do
           build_number = get_build_number_from_plist(
                           target: ENV['TARGET_NAME'],
                           plist_build_setting_support: true,
                          )
           version_number = get_version_number(
                              target: ENV['TARGET_NAME'],
                             )
           gym(scheme: ENV['SCHEME_NAME'], 
               configuration: "Release",
               output_directory: "./fastlane/build",
               output_name: ENV['SCHEME_NAME']+"-V"+version_number+"("+build_number+")",
               clean: true,
               export_method:"development")
       end
   
       desc "Deploy multi targets"
       lane :deploy_all do
           #cocoapods
           sh "fastlane deploy --env main"
           sh "fastlane deploy --env another"
       end
   end
   ```

6. 执行`fastlane deploy_all`

 写在暂时的最后，这玩意儿玩法很多，如果需要，可以继续更新此文档。


