---
title: jenkins实现持续集成
date: 2018-02-11 14:55:56
tags: [jenkins,继续集成]
---

## 集成环境

* macBook 
* java环境 1.8
* tomcat 7.0
* xcode环境 9.0
* jenkins(.war) v2.6+

### java环境安装

1.从orcale官网下载macOS对应的java版本进行下载

建议下载jdk-8u201-macosx-x64_bin.dmg

2.双击下载文件，继续双击出现图标

3.依次点击继续等安装完成，通过终端terminal，输入以下指令看是否完成

<!-- more -->

`java -version`

如果能正常显示java版本说明java安装成功

```
jjava version "1.8.0_201"
Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)
```

4.配置PATH和CLASS_PATH，打开终端输入以下指令，打开profile文件（需要sudo超级权限）

`sudo vim ~/.bash_profile`

按提示输入密码即可，正常情况vim编辑器会打开一个文件，在英文输入下，按“i”进入编辑模式，在文件最后添加一下内容：

```
JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk-1.8.0.jdk/Contents/Home"
 
CLASS_PATH="$JAVA_HOME/lib"
 
PATH=".:$PATH:$JAVA_HOME.bin"

```
添加完上述内容后，按‘esc’键退出编辑模式，输入‘：wq’保存并退出profile，需要配置生效，需要在终端执行如下指令：

`source ~/.bash_profile`


### tomcat安装

下载需要的版本进行解压，文件随便放哪里都可以执行的，我放在/User/用户/Library下面。

为了方便操作，把Tomcat的/bin路径放入环境变量里面，更新上面的.bash_profile文件，在最后添加

`export PATH=$JAVA_HOME/bin:$PATH:/Users/用户/Library/Tomcat7/bin`

然后完成后在终端执行以下指令让文件生效

`source ~/.bash_profile`

如果tomcat路径没有操作权限，可以通过以下命令分配权限

`sudo chmod 755 tomcat路径/bin/*.sh` 

终端中输入startup.sh启动tomat，启动完毕后，可以通过浏览器输入'http://loalhost:8080',成功可以看到tomcat页面


关闭tomcat可以在终端输入shutdown.sh


### xcode安装

xcode可以从mac的app store下载安装，具体就不说了。


### jenkins

下载需要的版本war包，放入之前Tomcat文件夹下/webapps文件夹中。

通过浏览器输入'http://loalhost:8080/jinkens'打开可以看到jinkens页面

第一次打开根据页面在输入指定路径下文件密码进行激活，安装进入安装页面，一般就选推荐安装，选择需要的插件安装，如果插件安装失败可以在jinkens上手动安装。

安装完需要设置第一个管理员账号，设置完就进入jinkens界面

插件安装，点击系统管理>管理插件>，这里就不逐个说明了，每个插件点进去都有介绍，下面的插件列表没有的在可选插件里面搜索出来，选中安装完重启jenkins就可以了
比如github、xcode、git、Gradle、NodeJs、Keychains and ProvisioningProfiles Mangement、Persistent Parameter 、PostBuildScript等插件


## 项目配置

通过jenkins新建任务，选择构建一个自由风格的项目，点击ok。

![Git分支参数和自定义环境参数](/images/jinkens/jinkens_01.jpg)

![Git仓库参数设置](/images/jinkens/jinkens_02.jpg)

![构建脚本和构建后脚本配置](/images/jinkens/jinkens_03.jpg)



## 打包脚本

iOS打包脚本

```
#! /bin/bash

#工程名
projectnmae=$1
#targetName
targetname=$2
# 导出路径
exportpath=$3
#打包模式
mode=$4


# 是否 传入工程名
if [ -z "$projectnmae" ]; then
    echo '${projectnmae} : 无效' 
    echo "请将projectNmae作为脚本第一个参数传入"; exit 1;
fi

# 是否 传入target名称
if [ -z "$targetname" ]; then
    echo '${projectnmae} : 无效' 
    echo "请将projectNmae作为脚本第一个参数传入"; exit 1;
fi




#工程绝对路径
project_path=${PWD}

echo '项目路径===='$project_path

#工程名

project_name=${projectnmae}

#打包模式 Debug/Release
development_mode=${mode}

#scheme名
scheme_name=${targetname}

#build文件夹路径
build_path=${project_path}/build

#plist文件所在路径
exportOptionsPlistPath=${project_path}/ExportOptions.plist
DistributionSPlistPath=${project_path}/DistributionSummary.plist

#导出.ipa文件所在路径
if [ -z "$exportpath" ]; then
    exportFilePath=${project_path}/ipa/${development_mode}
else  
    exportFilePath=${exportpath}
fi  

echo '导出文件路径 ==== ' ${exportFilePath}


echo '*** 正在 清理工程 ***'
xcodebuild \
clean -configuration ${development_mode} -quiet  || exit 
echo '*** 清理完成 ***'



echo '*** 正在 编译工程 For '${development_mode}
security unlock-keychain -p ${KEYCHAIN_PASSWORD} ~/Library/Keychains/login.keychain-db
#security unlock-keychain -p ${KEYCHAIN_PASSWORD} ${KEYCHAIN_PATH}
echo ${project_path}/${project_name}.xcworkspace
xcodebuild \
archive -workspace ${project_path}/${project_name}.xcworkspace \
-scheme ${scheme_name} \
-configuration ${development_mode} \
-archivePath ${build_path}/${project_name}.xcarchive -quiet  || exit
echo '*** 编译完成 ***'



echo '*** 正在 打包 ***'

if [ "${development_mode}" == "Debug" ];then
  echo ${exportOptionsPlistPath}
  xcodebuild -exportArchive -archivePath ${build_path}/${project_name}.xcarchive \
  -configuration ${development_mode} \
  -exportPath ${exportFilePath} \
  -exportOptionsPlist ${exportOptionsPlistPath} \
  -quiet || exit
elif [ "${development_mode}" == "Release" ];then
  echo ${DistributionSPlistPath}
  xcodebuild -exportArchive -archivePath ${build_path}/${project_name}.xcarchive \
  -configuration ${development_mode} \
  -exportPath ${exportFilePath} \
  -exportOptionsPlist ${DistributionSPlistPath} \
  -quiet || exit
else
  echo '***没有plist***'
fi


if [ -e $exportFilePath/$scheme_name.ipa ]; then
    # mv $exportFilePath/$scheme_name.ipa $exportFilePath/$scheme_name'_'${development_mode}'_'`date +%Y%m%d-%H/%M`.ipa
    #unzip $exportFilePath/$scheme_name.ipa  #解压ipa文件
    mv /Users/macserver/jenkinsconfig/workspace/CLS.ipa /Users/macserver/jenkinsconfig/workspace/CheDai-iOS/
    echo "*** .ipa文件已导出 ***"
    #open $exportFilePath
else
    echo "*** 创建.ipa文件失败 ***"
fi
echo '*** 打包完成 ***'
```


打包后钉钉群通知脚本

```
posturl="https://oapi.dingtalk.com/robot/send?access_token=xxxxx"

a=${BUILD_ENV}
b=${a/Debug/测试环境}
c=${b/Release/正式环境}

postdata=`cat << EOF
{
"msgtype": "text",
"text": {
"content": "【$c】CLS工程iOS打包失败！！"
},
"at": {
"atMobiles": [
"xxxxxx"
],
"isAtAll": "false"
}
}
EOF
`
curl -H "Content-Type:application/json" -X POST -d "$postdata"  $posturl
```