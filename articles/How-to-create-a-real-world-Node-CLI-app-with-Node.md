> * 原文地址：[How to create a real-world Node CLI app with Node](https://medium.freecodecamp.org/how-to-create-a-real-world-node-cli-app-with-node-391b727bbed3)
> * 原文作者：[Timber.io](https://medium.freecodecamp.org/@timberdotio)
> * 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
> * 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/How-to-create-a-real-world-Node-CLI-app-with-Node.md](https://github.com/dawn-teams/translate/blob/master/articles/How-to-create-a-real-world-Node-CLI-app-with-Node.md)
> * 译者：[靖鑫](https://github.com/luckyjing)
> * 校对者：[]()

---

## 使用 Node 构建命令行应用

![](https://img.alicdn.com/tfs/TB1a13bwTmWBKNjSZFBXXXxUFXa-1560-1024.png)

在 `JavaScript` 的开发领域内，命令行应用还尚未获得足够的关注度。事实上，大部分开发工具都应该提供命令行界面来给像我们一样的开发者使用，并且用户体验应该与精心创建的 Web 应用程序相当，比如一个漂亮的设计，有用的菜单，清晰的错误反馈，加载提示和进度条等。

目前并没有太多的实际教程来指导我们使用 `Node` 构建命令行界面，所以本文将是开篇之作，基于一个基本的 `hello world` 命令应用，逐步构建一个名为 `outside-cli` 的应用，它可以预测未来 10 天任何地方的天气情况。

![](https://img.alicdn.com/tfs/TB15dfVwQ7mBKNjSZFyXXbydFXa-754-499.png)

**提示**：有不少的库可以帮助你构建复杂的命令行应用，例如 [oclif](https://github.com/oclif/oclif)，[yargs](https://github.com/yargs/yargs) 和 [commander](https://github.com/tj/commander.js)，但是为了你更好地理解背后的原理，我们会尽可能保持外部依赖足够少。当然，我们假设你已经拥有了 `JavaScript` 和 `Node` 的基础知识。

## 入门

与其他的 `JavaScript` 项目一样，最佳实践便是创建 `package.json` 和一个空的入口文件，目前还不需要任何依赖，保持简单。

**package.json**

    {
      "name": "outside-cli",
      "version": "1.0.0",
      "license": "MIT",
      "scripts": {},
      "devDependencies": {},
      "dependencies": {}
    }

**index.js**

    module.exports = () => {
      console.log('Welcome to the outside!')
    }

我们将使用 `bin` 文件来运行这个新程序，并且会把 `bin` 文件添加到系统目录里，使其在任何地方都可以被调用。

    #!/usr/bin/env node
    require('../')()

是不是之前从未见过 `#!/usr/bin/env node` ? 它被称为 [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix))。它指明了执行这个脚本的解释程序，而非 `shell` 。

`bin` 文件需要保持简单，因为它的本意仅是用来调用主函数，我们所有的代码都应当放置在此文件之外，这样才可以保证模块化和可测试，同时也可以实现未来在其他的代码里被调用。

为了能够直接运行 `bin` 文件，我们需要赋予正确的文件权限，如果你是在 `UNIX` 环境下，你只需要执行 `chomd +x bin/outside` 便可以，`Windows` 用户就只能靠自己了，建议使用 `Linux` 子系统。

接下来，我们将添加 `bin` 文件到 `package.json` 里，随后当我们全局安装此包时( `npm install -g outside-cli` )，`bin` 文件会被自动添加到系统目录内。

**package.json**

    {
      "name": "outside-cli",
      "version": "1.0.0",
      "license": "MIT",
      "bin": {
        "outside": "bin/outside"
      },
      "scripts": {},
      "devDependencies": {},
      "dependencies": {}
    }

现在我们输入 `./bin/outside` ，就可以直接运行了，欢迎消息将会被打印出来，紧接着执行 `npm link` ，它将会在系统路径和你的二进制文件之间建立软连接，这样 `outside` 命令便可以在任何地方运行了。

命令行运行命令由参数和指令构成，参数（或「标志」）是指前缀为一个或两个连字符构成的值（例如  `-d`，`--debug` 或 `--env production` ），它对应用来说非常有用。指令是除了命令以外的其他值，它没有任何标志。

与指令不同，参数并不要求特定的顺序，举个例子，运行 `outside today Brooklyn`，必须约定第二个指令只能代表地域，使用 `--` 则不然，运行 `outside today --location Brooklyn`，可以方便地添加更多的选项。

为了使应用更加实用，我们需要解析指令和参数，然后转换为字面量对象，我们可以使用 `process.argv` 来手动实现，但是现在我们要安装项目的第一个依赖 [minimist](https://github.com/substack/minimist) ，让它来帮我们搞定这些事儿。

    npm install --save minimist

**index.js**

    const minimist = require('minimist')
    
    module.exports = () => {
      const args = minimist(process.argv.slice(2))
      console.log(args)
    }

**提示**：因为 `process.argv` 的前两个参数分别是解释器和二进制文件名，所以我们使用 `.slice(2)` 移除掉前两个参数，只关心传递进来的其他命令。

现在执行 `outside today` 将会输出 `{ _: ['today'] }`。执行 `outside today --location "Brooklyn, NY"`，将会输出 `{ _: ['today'], location: 'Brooklyn, NY' }`。不过现在我们不用进一步深挖参数的用法，等到实际使用 `location`的时候再继续深入，目前了解的已经足够我们实现第一个指令了。

## 参数语法

可以通过[这篇文章](https://www.gnu.org/software/libc/manual/html_node/Argument-Syntax.html)帮助你更好地理解参数语法。基本上，一个参数可以有一个或者两个连字符，然后紧跟着是它对应的值，在不填写时它的值默认为 `true`, 单连字符参数还可以使用缩写的格式（ `-a -b -c` 或者 `-abc` 都对应着 `{ a: true, b: true, c: true }` ）。

**如果参数值包含特殊字符或者空格，则必须使用引号包裹着**。例如 `--foo bar` `{ : ['baz'], foo: 'bar' }`，`--foo "bar baz"` 对应 `{ foo: 'bar baz' }`。

分割每个指令的代码，在其被调用时再加载至内存是一个最佳实践，这有助于缩短启动时间，避免不必要的加载。在主指令代码里简单地使用 `switch` 就可以实现此实践了。在这种设置下，我们需要把每个指令写到独立的文件里，并且导出一个函数，与此同时，我们把参数传递给每个指令函数用以在后期使用。

**index.js**

    const minimist = require('minimist')
    
    module.exports = () => {
      const args = minimist(process.argv.slice(2))
      const cmd = args._[0]
    
      switch (cmd) {
        case 'today':
          require('./cmds/today')(args)
          break
        default:
          console.error(`"${cmd}" is not a valid command!`)
          break
      }
    }

**cmds/today.js**

    module.exports = (args) => {
      console.log('today is sunny')
    }

现在如果执行 `outside today`，你会看到输出 `today is sunny`，如果执行 `outside foobar`，会输出 `"foobar" is not a valid command`。目前的原型已经很不错了，接下来我们需要添加天气的API来获取真实数据。

每个命令行应用里都应该包含一些参数：`help`，`--help` 和 `-h` 用来展示帮助清单；`--version` 和 `-v` 用来显示当前应用的版本信息。当指令没有指定时，我们也应当默认展示帮助清单。

`Minimist` 会自动解析参数为键值对，因此运行 `outside --version` 会使得 `args.version` 等于 `true`。那么在程序里通过设置 `cmd` 变量来保存 `help` 和 `version` 参数的判定结果，然后在 `switch` 语句中添加两个处理语句，就可以实现上述功能了。

    const minimist = require('minimist')
    
    module.exports = () => {
      const args = minimist(process.argv.slice(2))
    
      let cmd = args._[0] || 'help'
    
      if (args.version || args.v) {
        cmd = 'version'
      }
    
      if (args.help || args.h) {
        cmd = 'help'
      }
    
      switch (cmd) {
        case 'today':
          require('./cmds/today')(args)
          break
    
        case 'version':
          require('./cmds/version')(args)
          break
    
        case 'help':
          require('./cmds/help')(args)
          break
    
        default:
          console.error(`"${cmd}" is not a valid command!`)
          break
      }
    }

实现新指令时，格式需要和 `today` 指令保持一致。

**cmds/version.js**

    const { version } = require('../package.json')
    
    module.exports = (args) => {
      console.log(`v${version}`)
    }

**cmds/help.js**

    const menus = {
      main: `
        outside [command] <options>
    
        today .............. show weather for today
        version ............ show package version
        help ............... show help menu for a command`,
    
      today: `
        outside today <options>
    
        --location, -l ..... the location to use`,
    }
    
    module.exports = (args) => {
      const subCmd = args._[0] === 'help'
        ? args._[1]
        : args._[0]
    
      console.log(menus[subCmd] || menus.main)
    }

现在如果执行 `outside help today` 或 `outside toady -h`，你便会看到 `today` 指令的帮助信息了，执行 `outside` 或 `outside -h` 亦是如此。

![](https://img.alicdn.com/tfs/TB1b5PAwRjTBKNjSZFDXXbVgVXa-754-499.png)

目前的项目设定是令人愉悦的，因为当你需要添加一个新指令时，你只需要创建一个新指令文件，把它添加到 `switch` 语句中，再设置一个帮助信息便可以了。

**cmds/forecast.js**

    module.exports = (args) => {
      console.log('tomorrow is rainy')
    }

**index.js**

    *// ...*
        case 'forecast':
          require('./cmds/forecast')(args)
          break
    *// ...*

**cmds/help.js**

    const menus = {
      main: `
        outside [command] <options>
    
        today .............. show weather for today
        forecast ........... show 10-day weather forecast
        version ............ show package version
        help ............... show help menu for a command`,
    
      today: `
        outside today <options>
    
        --location, -l ..... the location to use`,
    
      forecast: `
        outside forecast <options>
    
        --location, -l ..... the location to use`,
    }
    
    // ...

有些指令执行起来可能需要很长时间。如果你会执行从 `API` 获取数据，内容生成，将文件写入磁盘，或者其他需要花费超过几毫秒的程序，那么便需要向用户提供一些反馈来表明你的程序仍在响应中。你可以使用进度条来展示操作的进度，也可以直接显示一个进度指示器。

对当前的应用来说，我们无法获知 `API` 请求的进度，所以我们使用一个简单的 `spinner` 来表达程序仍在运行中就可以了。我们接下来安装两个依赖，`axios` 用于网络请求，`ora` 来实现 `spinner`。

    npm install --save axios ora

## 从 API 获取数据

现在我们先创建一个使用雅虎天气 API 来获得某个地域天气情况的工具函数。

**提示**：雅虎 API 使用非常简洁的 `YQL` 语法，我们不需要刻意理解它，直接拷贝使用即可。另外，它也是唯一一个我发现不需要提供 `API key` 的天气 API 了。

**utils/weather.js**

    const axios = require('axios')
    
    module.exports = async (location) => {
      const results = await axios({
        method: 'get',
        url: 'https://query.yahooapis.com/v1/public/yql',
        params: {
          format: 'json',
          q: `select item from weather.forecast where woeid in
            (select woeid from geo.places(1) where text="${location}")`,
        },
      })
    
      return results.data.query.results.channel.item
    }

**cmds/today.js**

    const ora = require('ora')
    const getWeather = require('../utils/weather')
    
    module.exports = async (args) => {
      const spinner = ora().start()
    
      try {
        const location = args.location || args.l
        const weather = await getWeather(location)
    
        spinner.stop()
    
        console.log(`Current conditions in ${location}:`)
        console.log(`\t${weather.condition.temp}° ${weather.condition.text}`)
      } catch (err) {
        spinner.stop()
    
        console.error(err)
      }
    }

现在当你执行 `outside today --location "Brooklyn, NY"` 后，你首先会看到一个快速旋转的 `spinner` 出现在应用发起请求期间，随后便会展示天气信息了。

当请求发生得很快时，我们是难以看到加载指示的，如果你想人为地减慢速度，你可以在请求天气工具函数前加上这一句：`await new Promise(resolve => setTimeout(resolve, 5000))`。

![](https://img.alicdn.com/tfs/TB1w5ZfwOMnBKNjSZFCXXX0KFXa-637-209.gif)

非常棒！接下来我们复制下上面的代码来实现 `forecast` 指令，然后简单修改下输出格式。

**cmds/forecast.js**

    const ora = require('ora')
    const getWeather = require('../utils/weather')
    
    module.exports = async (args) => {
      const spinner = ora().start()
    
      try {
        const location = args.location || args.l
        const weather = await getWeather(location)
    
        spinner.stop()
    
        console.log(`Forecast for ${location}:`)
        weather.forecast.forEach(item =>
          console.log(`\t${item.date} - Low: ${item.low}° | High: ${item.high}° | ${item.text}`))
      } catch (err) {
        spinner.stop()
    
        console.error(err)
      }
    }

现在当你执行 `outside forecast --location "Brooklyn, NY"` 后，你会看到未来 10 天的天气预测结果了。接下来我们再锦上添花下，当 `location` 没有指定时，使用我们编写的一个工具函数来实现自动根据 IP 地址获取所处位置。

**utils/location.js**

    const axios = require('axios')
    
    module.exports = async () => {
      const results = await axios({
        method: 'get',
        url: 'https://api.ipdata.co',
      })
    
      const { city, region } = results.data
      return `${city}, ${region}`
    }

**cmds/today.js** & **cmds/forecast.js**

    *// ...*
    const getLocation = require('../utils/location')
    
    module.exports = async (args) => {
      *// ...*
        const location = args.location || args.l || await getLocation()
        const weather = await getWeather(location)
      *// ...*
    }

现在当你不添加 `location` 参数执行指令后，你将会看到当前地域对应的天气信息。

![](https://img.alicdn.com/tfs/TB1LsoawTCWBKNjSZFtXXaC3FXa-635-273.gif)

## 错误处理

本篇文章我们并不会详细介绍错误处理的最佳方案（后面的教程里会介绍），但是最重要的是要记住使用正确的退出码。

如果你的命令行应用出现了严重错误，你应当使用 `process.exit(1)`，终端会感知到程序并未完全执行，此时便可以通过 CI 程序来对外通知。

接下来我们创建一个工具函数来实现当运行一个不存在的指令时，程序会抛出正确的退出码。

**utils/error.js**

    module.exports = (message, exit) => {
      console.error(message)
      exit && process.exit(1)
    }

**index.js**

    *// ...*
    const error = require('./utils/error')
    
    module.exports = () => {
      *// ...*
        default:
          error(`"${cmd}" is not a valid command!`, true)
          break
      *// ...*
    }

## 收尾

最后一步是将我们编写的库发布到远程包管理平台上，由于我们使用 `JavaScript`，`NPM` 再合适不过了。现在，我们需要额外填一些儿信息到 `package.json` 里。

    {
      "name": "outside-cli",
      "version": "1.0.0",
      "description": "A CLI app that gives you the weather forecast",
      "license": "MIT",
      "homepage": "https://github.com/timberio/outside-cli#readme",
      "repository": {
        "type": "git",
        "url": "git+https://github.com/timberio/outside-cli.git"
      },
      "engines": {
        "node": ">=8"
      },
      "keywords": [
        "weather",
        "forecast",
        "rain"
      ],
      "preferGlobal": true,
      "bin": {
        "outside": "bin/outside"
      },
      "scripts": {},
      "devDependencies": {},
      "dependencies": {
        "axios": "^0.18.0",
        "minimist": "^1.2.0",
        "ora": "^2.0.0"
      }
    }

* 设置 `engine` 可以确保使用者拥有一个较新的 `Node` 版本。因为我们未经编译直接使用了 `async/await`，所以我们要求 `Node` 版本 必须在 8.0 及以上。

* 设置 `preferGlobal` 将会在安装时提示使用者本库最好全局安装而非作为局部依赖安装。

目前就这些内容了，现在你便可以通过 `npm publish` 发布至远端来供他人下载了。如果你想更进一步，发布到其他包管理工具（例如 `Homebrew` ）上，你可以了解下 [pkg](https://github.com/zeit/pkg) 或 [nexe](https://github.com/nexe/nexe)，它们可以帮助你把应用打包到一个独立的二进制文件里。

## 总结

本篇文章介绍的代码目录结构是 [Timber](https://timber.io/) 上所有的命令行应用都遵循的，它有助于保持组织和模块化。

对于速读的读者，我们也提供了一些本教程的**关键要点**：

* `Bin` 文件是整个命令行应用的入口，它的职责仅是调用主函数。

* 指令文件在未执行时不应该被加载到主函数里。

* 始终包含 `help` 和 `version` 指令。

* 指令文件需要保持简单，它们的主要职责是调用其他工具函数，随后展示信息给用户。

* 始终包含一些运行指示给到用户。

* 应用退出时应当使用正确的退出码。

<<<<<<< HEAD
我希望你现在能够更好地了解如何使用 `Node` 创建和组织命令行应用。本文只是开篇之作，随后我们会继续深入理解如何优化设计，生成 `ascii art` 和添加色彩等。本文的源码可以在 [GitHub](https://github.com/timberio/outside-cli) 上获取到。
=======
我希望你现在能够更好地了解如何使用 `Node` 创建和组织命令行应用。本文只是开篇之作，随后我们会继续深入理解如何优化设计，生成 `ascii art` 和添加色彩等。本文的源码可以在 [GitHub](https://github.com/timberio/outside-cli) 上获取到。
>>>>>>> 翻译How-to-create-a-real-world-Node-CLI-app-with-Node 完成

阿里云翻译小组