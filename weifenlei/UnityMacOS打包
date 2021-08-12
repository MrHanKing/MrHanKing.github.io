#  先说结果

* M1之后 基于公证系统之后的调整 此次试验没有成功。在出包的机子上软件运行无问题，出包机是M1。软件包打包成功、签名成功、公证成功。
但无法在其他机器上运行。在其他M1机器上提示【您没有权限来打开应用程序XXX，联系管理员XXXXX等等】，在非M1机器上运行，程序奔溃，提示【EXC_CRASH (Code Signature Invalid)】。
* 建议: 用unity出完MacOS包后 走xcode的 Mac Apple Store流程。

# 过程

## unity出包

* 正常出包流程 build构建

## 签名

* 根据需求在开发者后台准备2个证书，一个application证书，一个installer证书。
* 准备.entitlements用于签名的时候 配置各种权限配置。可以从provisionprofile文件中提取出来。
  ```
  security cms -D -i XXX.provisionprofile > temp.plist
  /usr/libexec/PlistBuddy -x -c 'Print :Entitlements' temp.plist > entitlements.plist
  ```
* 给予app修改权限
  ```
  chmod -R a+xr app全路径名字.app
  ```
* 本地创建.sh脚本，对XXX.app内所有二进制文件进行application证书签名
  ```
  BINARY=$1
  ENTITLEMENTS=$2
  SIGNCERT=$3
  libfile=`find ./$BINARY -name '*.dylib'`
  while read -r libname; do
          echo "Sign:" $libname
          codesign --deep --force --verify -vvvv --timestamp --options runtime --entitlements  $ENTITLEMENTS --sign "$SIGNCERT" $libname
  done <<< "$libfile"

  echo "dylib ok"

  bundlefile=`find ./$BINARY -name '*.bundle'`
  while read -r bundlename; do
          echo "Sign:" $bundlename
          codesign --deep --force --verify -vvvv --timestamp --options runtime --entitlements  $ENTITLEMENTS --sign "$SIGNCERT" $bundlename
  done <<< "$bundlefile"

  echo "bundle ok"

  frameworkfile=`find ./$BINARY -name '*.framework'`
  while read -r frameworkname; do
          echo "Sign:" $frameworkname
          codesign --deep --force --verify -vvvv --timestamp --options runtime --entitlements  $ENTITLEMENTS --sign "$SIGNCERT" $frameworkname
  done <<< "$frameworkfile"

  echo "framework ok"
  ```
* 对.app文件进行签名
  ```
  codesign --deep --force --verify --verbose --timestamp --options runtime --entitlements "entitlements文件全路径" --sign "application证书名字" app的路径名字.app -i “bundleId的内容”
  ```
* 对.app构建pkg包（在公证之后进行 且构建的pkg还要公证）
  ```
  productbuild --component app全路径名字.app /Applications --sign "installer证书" pkg全路径名字.pkg 
  ```

## 公证

*  后台同意苹果官方的各种协议
*  把签名的待公证app打成zip包
```
/usr/bin/ditto -c -k --keepParent "xxxx.app " "xxxx.zip"
```
*  创建苹果官方文档说的AC_Access 方便后面使用。
```
xcrun altool --store-password-in-keychain-item "AC_KeyName" -u "apple id" -p "apple id非开发者平台登陆后生成的高级密钥"
```
*  获取自己的Providers, 拿到ProviderShortname
```
xcrun altool --list-providers -u "apple id" -p "@keychain:AC_KeyName" 
```

* 提交公证验证
```
xcrun altool --notarize-app --primary-bundle-id "开发者后台profile对应的Apple id" --username "apple id(就是账户邮箱)" --password "@keychain:AC_KeyName" --asc-provider "上面获取的拿到ProviderShortname" --file "需要公证的zip"
```

* 如果上一步成功, 等待2-3分钟 获取公证结果
```
xcrun altool --notarization-info "提交公证返回的RequestUUID" -u "apple id" -p "@keychain:AC_KeyName" 
```

* 给.app签入公证反馈
```
xcrun stapler staple 提交公证的app.app 
```

## 验证命令

* 查看签名情况
```
codesign --verify --deep -vvvv --strict app全路径名字.app
codesign --verify --deep --strict --verbose=4 app全路径名字.app 
```
* 查看公证情况
```
spctl -a -v app全路径名字.app
```
* 查看本地所有证书
```
xcrun security find-identity -v -p codesigning 
```
