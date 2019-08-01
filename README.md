创建视频聊天程序--使用OpenTok和React

## 官方教程：
https://www.nexmo.com/blog/2019/07/16/opentok-react-components-dr

## 前提条件
* Node.js(https://nodejs.org/en/)版本在5.2以上
* Tokbox账户 https://tokbox.com/account/user/signup

## 基本流程
* 创建Tokbox项目
* 创建React应用
* 创建可重用的React组建
** 发布者
** 订阅者
** 连接状态
** 选项卡
* 运行视频聊天程序

## 创建Tokbox项目

* 创建Tokbox账户
Projects -> Create New Project -> OpenTok API -> Create -> PROJECT_API_KEY
* 创建会话ID
Projects -> YOUR_PROJECT -> Project Tools -> Create Session ID -> SESSION_ID
* 生成令牌
SESSION_ID -> Generate Token -> GENERATED_TOKEN

以上大写字符表示的对象是系统生成的关键变量名称，需要保存和记录好备用。

## 创建React应用

创建React应用，名称可以自定义为其他（这里用`react-components-tokbox`）。进入项目目录并安装依赖的库。最后运行程序确认初始化无误。
```
$ npx create-react-app react-components-tokbox
$ cd react-components-tokbox && npm install opentok-react lodash
$ npm start
```
创建`src/config.js`文件，使用之前保存的信息替换这里的占位符`XYZ`:
```
export default {
  API_KEY: 'XYZ',
  SESSION_ID: 'XYZ',
  TOKEN: 'XYZ'
};
```
更新`src/index.js`文件，在导入的文件中追加刚才添加的配置文件，如下：
```
import config from './config';
```
继续更新，在默认的APP定义中追加对登陆信息变量的引用，如下：
```
# 以下为原始默认配置
ReactDOM.render(<App />, document.getElementById('root'));
# 以下为更新配置
ReactDOM.render(<App
  apiKey={config.API_KEY}
  sessionId={config.SESSION_ID}
  token={config.TOKEN}
/>, document.getElementById('root'));
```
更新`src/App.js`，添加OpenTok的相关引用：
```
import { OTSession, OTStreams, preloadScript } from 'opentok-react';
```

## 创建可重用React组件

在`src`目录创建以下字目录和文件：
```
$ mkdir components
$ touch Publisher.js Subscriber.js ConnectionStatus.js CheckBox.js
```
更新`src/App.js`文件，引用之前创建的文件：
```
import ConnectionStatus from './components/ConnectionStatus';
import Publisher from './components/Publisher';
import Subscriber from './components/Subscriber';
```
继续更新以上文件对`App`的定义，删除原来的函数定义和导出语句，为以下的类定义和导出语句：
```
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {};
  };
  render() {
    return (
      <div>
        TokBox
      </div>
    );
  }
}
export default preloadScript(App);
```
在以上更新的基础上继续更新构建函数：
```
constructor(props) {
  super(props);
  this.state = {
    error: null,
    connected: false
  };
  this.sessionEvents = {
    sessionConnected: () => {
      this.setState({ connected: true });
    },
    sessionDisconnected: () => {
      this.setState({ connected: false });
    }
  };
}
```
继续更新，添加错误处理函数：
```
onError = (err) => {
  this.setState({ error: `Failed to connect: ${err.message}` });
}
```
继续，在`render()`函数中添加`<OTSession />`组件从`src/index.js`中导入登录凭据：
```
render() {
  return (
    <OTSession
      apiKey={this.props.apiKey}
      sessionId={this.props.sessionId}
      token={this.props.token}
      eventHandlers={this.sessionEvents}
      onError={this.onError}
      >
    </OTSession>
  );
}
```
继续，在`<OTSession />`组件的内部添加错误处理及其他组件：
```
  {this.state.error ? <div id="error">{this.state.error}</div> : null}
  <ConnectionStatus />
  <Publisher />
  <OTStreams>
    <Subscriber />
  </OTStreams>
```

## ConnectionStatus组件

更新`src/components/ConnectionStatus.js`文件，显示连接状态：
```
import React from 'react';
 
class ConnectionStatus extends React.Component {
  render() {
    let status = this.props.connected ? 'Connected' : 'Disconnected';
    return (
      <div className="connectionStatus">
        <strong>Status:</strong> {status}
      </div>
    );
  }
}
export default ConnectionStatus;
```
更新`src/App.js`文件：
```
<ConnectionStatus connected={this.state.connected} />
```

## Publisher组件

更新`src/components/Publisher.js`文件，导入必要的依赖项目：
```
import React from 'react';
import { OTPublisher } from 'opentok-react';
import CheckBox from './CheckBox';
```
继续，创建`Publisher`类：
```
class Publisher extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      error: null,
      audio: true,
      video: true,
      videoSource: 'camera'
    };
  }
  setAudio()
  setVideo()
  setVideoSource()
  onError()

  render() {
    return (
      <div>
        <OTPublisher />
      </div>
    );
  }
}
export default Publisher;
```
继续，完善类定义中的内建方法：
```
setAudio = (audio) => {
  this.setState({ audio });
}

setVideo = (video) => {
  this.setState({ video });
}

changeVideoSource = (videoSource) => {
  (this.state.videoSource !== 'camera') ? this.setState({videoSource: 'camera'}) : this.setState({ videoSource: 'screen' })
}
 
onError = (err) => {
  this.setState({ error: `Failed to publish: ${err.message}` });
}
```
继续，更新`render()`函数，在`OTPublisher`组件属性中添加`publishAudio`，`publishVideo`，和 `videoSource`，添加`onError()`函数到`onError`属性：
```
render() {
  return (
    <div className="publisher">
      Publisher
      {this.state.error ? <div id="error">{this.state.error}</div> : null}
      <OTPublisher
        properties={{
          publishAudio: this.state.audio,
          publishVideo: this.state.video,
          videoSource: this.state.videoSource === 'screen' ? 'screen' : undefined
        }}
        onError={this.onError}
      />
    </div>
  )
};
```

## Subscriber组件

在`src/components/Subscriber.js`文件中，导入必要的依赖项目，创建`Subscriber `类对象，带有属性`ubscribeToAudio`和`subscribeToVideo`，就像`OTSubscriber`的属性一样。
```
import React from 'react';
import { OTPublisher } from 'opentok-react';
import CheckBox from './CheckBox';

class Publisher extends React.Component {
    constructor(props) {
      super(props);
      this.state = {
        error: null,
        audio: true,
        video: true,
        videoSource: 'camera'
      };
    }

    setAudio = (audio) => {
        this.setState({ audio });
    }
     
    setVideo = (video) => {
        this.setState({ video });
    }
     
    changeVideoSource = (videoSource) => {
        (this.state.videoSource !== 'camera') ? this.setState({videoSource: 'camera'}) : this.setState({ videoSource: 'screen' })
    }
     
    onError = (err) => {
        this.setState({ error: `Failed to publish: ${err.message}` });
    }
  
    render() {
        return (
          <div className="publisher">
            Publisher
            {this.state.error ? <div id="error">{this.state.error}</div> : null}
            <OTPublisher
              properties={{
                publishAudio: this.state.audio,
                publishVideo: this.state.video,
                videoSource: this.state.videoSource === 'screen' ? 'screen' : undefined
              }}
              onError={this.onError}
            />
          </div>
        )
    };  
}

export default Publisher;
```

## CheckBox组件

更新`src/components/CheckBox.js`文件定义的`CheckBox`组件会用在`Publisher`和`Subscriber`组件中：
```
import React from 'react';
import { uniqueId } from 'lodash';
 
class CheckBox extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      id: uniqueId('Checkbox'),
      isChecked: this.props.initialChecked
    };
  }
 
  onChange = (event) => {
    let isChecked = event.currentTarget.checked;
    this.setState({ isChecked });
  }
 
  componentDidUpdate(prevProps, prevState) {
    if (
      prevState.isChecked !== this.state.isChecked &&
      typeof this.props.onChange === 'function'
    ) {
      this.props.onChange(this.state.isChecked);
    }
  }
 
  render() {
    return (
      <div>
        <label htmlFor={this.state.id}>
          {this.props.label}
        </label>
        <input
          type="checkbox"
          checked={this.state.isChecked}
          id={this.state.id}
          onChange={this.onChange}
        />
      </div>
    );
  }
}
export default CheckBox;
```
这里关于`CheckBox`组件的应用，原教程没有说清楚，但从完成的源码可以看到，相关的应用代码部署到了`Publisher`和`Subscriber`组件的`render()`函数所返回的`<div>`HTML组件中，跟在`OTPublisher`组件后面。
如上，更新`src/components/Publisher.js`文件：
```
  render() {
    return (
      <div className="publisher">
        Publisher

        {this.state.error ? <div id="error">{this.state.error}</div> : null}

        <OTPublisher
          properties={{
            publishAudio: this.state.audio,
            publishVideo: this.state.video,
            videoSource: this.state.videoSource === 'screen' ? 'screen' : undefined
          }}
          onError={this.onError}
        />

        <CheckBox
          label="Share Screen"
          onChange={this.changeVideoSource}
        />

        <CheckBox
          label="Publish Audio"
          initialChecked={this.state.audio}
          onChange={this.setAudio}
        />

        <CheckBox
          label="Publish Video"
          initialChecked={this.state.video}
          onChange={this.setVideo}
        />

      </div>
    );
  }
```
更新`src/components/Subscriber.js`文件：
```
  render() {
    return (
      <div className="subscriber">
        Subscriber
        {this.state.error ? <div id="error">{this.state.error}</div> : null}
        <OTSubscriber
          properties={{
            subscribeToAudio: this.state.audio,
            subscribeToVideo: this.state.video
          }}
          onError={this.onError}
        />

        <CheckBox
          label="Subscribe to Audio"
          initialChecked={this.state.audio}
          onChange={this.setAudio}
        />
        <CheckBox
          label="Subscribe to Video"
          initialChecked={this.state.video}
          onChange={this.setVideo}
        />
      </div>
    );
  }
```

## 装点

更新`src/App.css`文件：
```
body, html {
  background-color: pink;
  height: 100%;
  font-family: Verdana, Geneva, sans-serif
}
.connectionStatus {
  padding-top: 20px;
  margin-left: 5%;
  font-size: 1.5em;
}
.subscriber {
  margin-left: 10%;
}
.publisher {
  float: right;
  margin-right: 10%;
}
.OTPublisherContainer {
  width: 80vh !important;
  height: 80vh !important;
}
#error {
  color: red;
}
```

## 测试

运行服务器并在网页中查看效果：
```
$ npm start
```
问题是，可以看到Publisher的视频，但看不到Subscriber的视频。也就是在教程的测试结果页面上，可以看到右侧的图像但看不到左侧的图像。从官方代码库客隆程序到本地运行，结果也是一样。如果要运行克隆的程序，先运行`npm install`。

## 参考
* 完成的代码
https://github.com/nexmo-community/react-components-tokbox
