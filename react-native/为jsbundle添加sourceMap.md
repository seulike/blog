## 前言  
一般react-native项目中会使用sentry或者接入友盟来获取报错日志。
由于rn中打包后的js都是压缩过的代码，这里面的报错信息行列的定位都是对于压缩后的代码。
因而需要sourceMap来将错误定位到项目的源代码的位置。rn一般默认没有sourceMap，因而需要做一定的设置
来生成。[stackoverflow上面也有相关讨论](https://stackoverflow.com/questions/34715106/how-to-add-sourcemap-in-react-native-for-production)
这里面的方法还需要稍加调整，才能最终得以生效。下面将介绍具体的配置方法。
通过的本文的配置，最中会在项目目录下生成文件sourceMap/sourcemap.android.js，
sourcemap/sourcemap.ios.js，分别为android和ios的sourcemap。
首先在项目目录下创建文件sourceMap。

### android配置
android的配置方法比较简单。只需在项目的android/app/build.gradle中
找到project.ext.react，添加extraPackagerArgs
```
project.ext.react = [
        entryFile: "index.js",
        extraPackagerArgs: ["--sourcemap-output", file("$buildDir/../../../sourceMap/sourcemap.android.js")]
]
```

### ios配置
在sourceMap目录下创建一个空文件null_config(touch null_config).
在项目的配置中，找到Build Phases/Bundle React Native code and image
改变里面的shell脚本：
```
export NODE_BINARY=node
export BUNDLE_CONFIG="./sourceMap/null_config --sourcemap-output ./sourceMap/sourcemap.ios.js"
../node_modules/react-native/scripts/react-native-xcode.sh
```
原因在于，在打包jsbundle的时react-native-xcode.sh。sh中的打包命令是：
```
$NODE_BINARY $CLI_PATH $BUNDLE_COMMAND \
  $CONFIG_ARG \
  --entry-file "$ENTRY_FILE" \
  --platform ios \
  --dev $DEV \
  --reset-cache \
  --bundle-output "$BUNDLE_FILE" \
  --assets-dest "$DEST"
  ```
  会使用到参数$CONFIG_ARG：
  ```
  if [[ -z "$BUNDLE_CONFIG" ]]; then
  CONFIG_ARG=""
else
  CONFIG_ARG="--config $(pwd)/$BUNDLE_CONFIG"
fi
```
因此可以巧妙的把--sourcemap-output参数加到$BUNDLE_CONFIG中，从而加到了$BUNDLE_COMMAND的参数中。
当然完全也可以在react-native-xcode.sh脚本中直接把--sourcemap-output参数加入进来。


## 查找代码位置
通过上面的配置方法，以后在对android和ios打包的时候会在项目目录的sourceMap下生成对应的sourcemap文件。
如果在sentry中，有项目的报错日志。获取相应的行列，需要找到源代码中的位置。可以编写一下脚本。
在sourceMap中添加source.js：
```
const sourceMap = require('source-map')
const fs = require('fs')
const path = require('path')
const args = process.argv.slice(2)
const [line, column, platform] = args
const sourceFile =
    platform === 'ios' ? './sourcemap.ios.js' : './sourcemap.android.js'
async function getPosition(line, column) {
    const consumer = await new sourceMap.SourceMapConsumer(
        fs.readFileSync(path.join(__dirname,sourceFile), 'utf8')
    )
    console.log(
        consumer.originalPositionFor({
            line,
            column,
        })
    )
}
getPosition(line, column)
```
在项目的npm scripts中添加："source": "node ./sourceMap/source.js"
这样可以运行yarn run source 16 29356 ios


