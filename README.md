# KMG FONT-END 2C 项目开发指南 #

### 技术栈
框架体系：
`react - v1.18.12 / redux / typescript - v4.0`

其他：
`webpack / scss / lodash / axios`

### 安装工程
`npm start`

### 启动工程
`npm run build` -> `编译/打包/开发模式` -> `开发模式` -> `对应项目-对应客户端`

### 开发/打包配置
参见 `/deployment/projects.config.js`

### 插件
请在VS CODE中安装以下插件，来激活团队IDE配置与格式化配置`Prettier`, `EditorConfig`

### 跨域
`默认开发时的API接口为：webpack - proxy`

### 基本目录结构
<pre>
web-main
  |- assets
  |- cli
  |- deployment
  |- src
  |   |- assets
  |   |- core (核心文件，影响全平台全端)
  |       |-actions     * redux行为
  |       |-apis        * 接口相关
  |       |-constants   * 常量（配置等）
  |       |-middleware  * redux中间件
  |       |-mocks       * 模拟接口数据，临时存放
  |       |-helpers     * 工具类
  |       |-i18n        * 国际化目录
  |       |-hooks       * React HOOK - 提供视图层主体逻辑
  |       |-reducers    * redux reducer
  |       |-schemas     * TS全局声明
  |       |-store       * redux store
  |       |-types       * redux types
  |       |-templates   * 公共模板
  |          |-mobile   * 手机端模板
  |          |-desktop  * PC端模板
  |          |-static   * 静态模板
  |       |
  |   |- views (视图文件，影响单个项目或单个端)
  |       |-kmg（视图工程）
  |         |-assets  * 静态资源
  |         |-public  * 公共配置 - 影响单个项目
  |         |-desktop * PC端视图
  |         |-mobile  * H5端视图
  :       :    :
</pre>

### 配置管理
为了最大程度复用和分离平台与端的耦合，配置采用三层配置并使用继承与复写来
- 公共配置：`core/constants/configs` - 该配置影响全平台
- 项目配置：`views/{platform}/public/configs.ts` - 影响某平台（不分端），在此配置中引用公共配置，并向下导出
- 客户端配置：`views/{platform}/{client}/configs.ts` - 影响单平台单端，在此配置中引用项目配置

例：全平台公共配置 DEFAULT_AGENT_CODE，但其中A平台有URL差异，则在【项目配置】中进行变量复写
```javascript
import CONFIG from '@constants/configs';

export const DEFAULT_AGENT_CODE = '000222'

export default {
  ...CONFIG,
 DEFAULT_AGENT_CODE
}
```
相同，如果只是客户端级别差异，则在【客户端配置】中进行复写

### 别名
对于不同工程的私有路径，如何在公共组件中进行区分，例如：
KMG前台平台配置文件路径：`src/views/kmg/desktop/config.ts`
KMG后台配置文件路径：`src/views/kmg-backend/desktop/config.ts`
在公共组件中要使用对应的配置文件就需要用到别名进行处理
```javascript

// projects.config.js
const kmg = {
  client: client === 'pc' ? 'desktop' : 'mobile',
  base: 'mc',
}

// webpack
const alias = {
  '@this': path.join(__dirname, `../src/views/${CONFIG.base}/${CONFIG.client}`)
}

// component
import CONFIGS from '@this/config'
```

### 接口缓存
对于接口数据缓存，添加了以下参数进行拓展：
```javascript
/**
 * 请求缓存相关
 * @param expires      缓存时间 单位 分钟
 * @param forward      当有缓存数据时，是否仍然请求去更新当前缓存
 * @param isUserBind   该缓存是否绑定当前用户
 * @param cacheClear   是否弃用当前缓存数据，重新缓存
 */
```
### 接口超时处理
接口超时处理如下图所示：
```javascript
/**
 * 请求超时相关
 * @param timeout         自定义超时时间 单位 s
 * @param timeoutCallback 自定义超时回调函数
 * @param maxRetry        最大重试次数
 */
```
![avatar](./assets/markdown/timeout.png)

### 页面白屏异常处理
对于页面/子组件在render过程中发成报错造成的白屏，针对多个项目在 `/views/$client/components/PageHOC` 中使用 class component 中 `componentDidCatch` 生命周期将其捕获，后经
`/views/$client/pages/Error` 将其错误信息进行友好抛出，核心代码如下：
```javascript
class ErrorHandler extends React.Component {

  constructor(props: any) {
    super(props);
    this.state = { hasError: false };
  }

  componentDidCatch(error: any) {
    this.setState({ hasError: error })
  }

  render() {
    if ((this.state as any).hasError) {
      return <ErrorScreen log={(this.state as any).hasError} />
    }
    return (this.props as any).children
  }
}
```

### React HOOK
React hook作为本次重构的核心，在解耦合中起了重要的作用，已将在class component中以下部分移植到HOOK中
- 所有HOC相关的操作
- 所有services的操作
- 不区分客户端的组件生命周期管理
- redux相关操作
- `aHook` 库作为常用的 hook 被添加使用  [官方文档](https://ahooks.js.org/guide)

### 样式管理
#### 目录
对于大多数【页面】/【组件】，在同一目录下都一一对应一个style scss文件相依赖，例如：
- 首页 `home/index.tsx` 在其同一目录中 `home/style.scss` 与之对应
- banner `template/desktop/components/banner/index.tsx` 在其同一目录中 `/style.scss` 与之对应

#### 主要的依赖结构
![avatar](./assets/markdown/scss.png)

#### 模块化
为了避免相同className导致不同组件间样式冲突，每个组件或者页面都使用css模块化进行root节点类命名
```scss
:local(.wrapper){

}
```
```javascript
import css from './style.scss';
return (
  <div className={css.wrapper}>
  </div>
)
```

#### 头部引用规范
对于依赖引用，应按照如下顺序从上之下进行引用
- ts定义文件 `import { TAction } from './_actionTypes.d.ts'`
- 第三方库 `import dayJs from 'dayjs'` / `import QR from 'qrcode.react'` etc..
- 逻辑层HOOK `import useLogin from '@hooks/useLogin'`
- 组件 `import Popup from '@this/shadow/Popup'` / `import Child from '../Child'` etc...
- 帮助/全局类 `import G from '@constants/global'` / `import { formatDate } from '@helpers/unit'` etc...
- 样式 `import css from './style.scss'`
一个相对完整的示例如下：
```javascript
import { TAction } from './_actionTypes.d.ts'
import { useParams } from 'react-router';
import usePromotion from "@core/hooks/usePromotion";
import usePublicState from "@core/hooks/usePublicState";
import PageHOC from "@this/components/PageHOC";
import { SkeletonLoader, NoData } from '@this/shadow/Loading';
import Banner from '@this/shadow/Banner';
import Select from '@this/shadow/Select';
import G from '@constans/global'
import css from './style.scss';
```

#### 文件注释规范
推荐使用VS Code头部注释插件：【koroFileHeader】并添加以下配置到 IDE SETTINGS 中
```JSON
// 头部注释
"fileheader.customMade": {
  "Author": "${your name}.GE",
  "Date": "Do not edit",
  "LastEditors": "${your name}.GE",
  "FilePath": "Do not edit",
  "Description": "",
},
// 函数注释
"fileheader.cursorMode": {
  "Author": "${your name}.GE",
  "Date": "Do not edit",
  "description": "",
  "param": "",
  "return": "",
},
```

#### CLI 子工程


#### 影子组件


#### 公共组件样式差异化


#### 公共组件结构（功能）差异化


#### redux store 状态差异化


#### 国际化


#### 开发新平台步骤


#### 约束（以下行为在开发过程中不允许出现）


#### 其他规范
# 2024-04-04
# 2024-04-04
[hotfix] : fixed 2993
[hotfix] : fixed 2993
[hotfix] : fixed 2993
[hotfix] : fixed 2993
[hotfix] : fixed 2993
[hotfix] : fixed 2993
[update] : updated 3024
[hotfix] : fixed 3019
[update] : updated 3018
[hotfix] : fixed 3027
[hotfix] : fixed 3041
[update] : updated 3022
[fix] : fixed odds-calculation user-balance
[update] : updated 3028
[update] : updated 3002
[feat] : fixed tournament-betting live-chat-support
[fix] : fixed system-performance user-experience
[update] : updated 3000
[feat] : fixed tournament-betting live-chat-support
[hotfix] : fixed 3019
[fix] : fixed match-schedule bet-history
[feat] : fixed referral-system personalized-recommendations
[feat] : fixed advanced-betting-options live-dealer-games
[feat] : fixed cash-out-feature live-streaming
[hotfix] : fixed 3019
[update] : updated 3022
[fix] : fixed system-performance user-experience
[update] : updated 3042
[feat] : fixed virtual-reality-experiences advanced-betting-strategies
[fix] : fixed user-feedback-system payment-security
[feat] : fixed voice-command-betting augmented-reality-betting
[feat] : fixed esports-betting loyalty-program
[hotfix] : fixed 3039
[fix] : fixed system-performance user-experience
[fix] : fixed data-integrity user-profile
[update] : updated 2994
[update] : updated 3034
[fix] : fixed user-authentication notification-system
[fix] : fixed system-performance user-experience
[hotfix] : fixed 2995
[hotfix] : fixed 3007
[update] : updated 3040
[fix] : fixed system-performance user-experience
[update] : updated 3028
[hotfix] : fixed 3029
[update] : updated 3030
[feat] : fixed game-result
[feat] : fixed esports-betting loyalty-program
[hotfix] : fixed 3025
[fix] : fixed user-authentication notification-system
[hotfix] : fixed 3017
[update] : updated 3040
[hotfix] : fixed 3013
[update] : updated 3042
[update] : updated 3020
[hotfix] : fixed 3005
[feat] : fixed artificial-intelligence-predictions social-betting-features
[update] : updated 3040
[hotfix] : fixed 3009
[update] : updated 3006
[update] : updated 3042
[hotfix] : fixed 3007
[update] : updated 3014
[hotfix] : fixed 3003
[update] : updated 3020
[feat] : fixed multi-language-support mobile-optimization
[fix] : fixed bet-slip-errors payment-processing
[feat] : fixed in-play-betting virtual-sports
[hotfix] : fixed 3023
[update] : updated 3032
[update] : updated 3018
[update] : updated 3014
[hotfix] : fixed 3031
[update] : updated 3040
[fix] : fixed bet-limits fraud-detection
[update] : updated 3012
[hotfix] : fixed 3007
[hotfix] : fixed 2997
[update] : updated 3000
[fix] : fixed account-verification deposit-methods
[fix] : fixed user-feedback-system payment-security
[feat] : fixed blockchain-integration gamification-features
[update] : updated 3012
[fix] : fixed account-verification deposit-methods
[hotfix] : fixed 3037
[update] : updated 3024
[feat] : fixed live-betting withdrawal-system
[feat] : fixed live-betting withdrawal-system
[hotfix] : fixed 3039
[fix] : fixed system-performance user-experience
[hotfix] : fixed 3033
[hotfix] : fixed 3033
[update] : updated 3022
[hotfix] : fixed 3003
[update] : updated 3008
[fix] : fixed data-integrity user-profile
[update] : updated 3000
[update] : updated 3002
[fix] : fixed system-performance user-experience
[hotfix] : fixed 2995
[update] : updated 3036
[fix] : fixed data-security bet-settlement
[hotfix] : fixed 3041
[fix] : fixed account-verification deposit-methods
[update] : updated 3008
[fix] : fixed user-authentication notification-system
[feat] : fixed game-result
[fix] : fixed system-performance user-experience
[update] : updated 2996
[fix] : fixed user-verification bet-slip-validation
[hotfix] : fixed 3001
[feat] : fixed mobile-app affiliate-program
[hotfix] : fixed 3023
[update] : updated 3026
[hotfix] : fixed 3035
[feat] : fixed cash-out-feature live-streaming
[feat] : fixed mobile-app affiliate-program
[hotfix] : fixed 3021
[update] : updated 3022
[hotfix] : fixed 3031
[feat] : fixed advanced-betting-options live-dealer-games
[feat] : fixed tournament-betting live-chat-support
[update] : updated 3000
[feat] : fixed live-betting withdrawal-system
[fix] : fixed customer-service user-feedback
[feat] : fixed artificial-intelligence-predictions social-betting-features
[fix] : fixed bet-slip-errors payment-processing
[update] : updated 3006
[feat] : fixed live-betting withdrawal-system
[update] : updated 3036
[hotfix] : fixed 3007
[fix] : fixed account-verification deposit-methods
[fix] : fixed user-account-security bet-slip-errors
[update] : updated 3010
[fix] : fixed bet-slip-management user-interface
[feat] : fixed blockchain-integration gamification-features
[update] : updated 3028
[feat] : fixed loyalty-rewards virtual-reality-betting
[fix] : fixed bet-cancellation account-management
[feat] : fixed live-betting withdrawal-system
[update] : updated 3026
[fix] : fixed user-authentication notification-system
[feat] : fixed multi-language-support mobile-optimization
[hotfix] : fixed 2999
[update] : updated 3014
[hotfix] : fixed 3003
[hotfix] : fixed 3019
[feat] : fixed live-betting withdrawal-system
[update] : updated 3014
[fix] : fixed user-feedback-system payment-security
[update] : updated 3016
[hotfix] : fixed 3021
[update] : updated 3016
[update] : updated 3034
[hotfix] : fixed 3015
[fix] : fixed user-interface-updates data-analysis
[fix] : fixed payment-gateway security-updates
[hotfix] : fixed 3023
[update] : updated 3016
[hotfix] : fixed 3019
[update] : updated 3002
[fix] : fixed customer-service user-feedback
[fix] : fixed bet-slip-errors payment-processing
[feat] : fixed mobile-app affiliate-program
[update] : updated 3026
[hotfix] : fixed 3005
[hotfix] : fixed 3003
[update] : updated 3002
[hotfix] : fixed 3029
[hotfix] : fixed 3007
[fix] : fixed bet-slip-errors payment-processing
[hotfix] : fixed 3013
[fix] : fixed match-schedule bet-history
[update] : updated 2994
[fix] : fixed user-interface-improvements system-stability
[feat] : fixed in-play-betting virtual-sports
[update] : updated 3004
[hotfix] : fixed 3021
[feat] : fixed esports-betting loyalty-program
[feat] : fixed advanced-betting-options live-dealer-games
[feat] : fixed esports-betting loyalty-program
[hotfix] : fixed 3013
[fix] : fixed user-authentication notification-system
[feat] : fixed promotional-offers social-media-integration
[update] : updated 3030
[fix] : fixed user-authentication notification-system
[hotfix] : fixed 3037
[update] : updated 3012
[hotfix] : fixed 3027
[fix] : fixed customer-service user-feedback
[fix] : fixed user-verification bet-slip-validation
[update] : updated 3022
[feat] : fixed live-betting withdrawal-system
[fix] : fixed account-verification deposit-methods
[fix] : fixed payment-gateway security-updates
[hotfix] : fixed 3037
[feat] : fixed virtual-reality-experiences advanced-betting-strategies
[hotfix] : fixed 3041
[hotfix] : fixed 3021
[fix] : fixed user-authentication notification-system
[hotfix] : fixed 3011
[update] : updated 3016
[feat] : fixed mobile-app affiliate-program
[hotfix] : fixed 3021
[hotfix] : fixed 3015
[feat] : fixed in-play-betting virtual-sports
[fix] : fixed account-verification deposit-methods
[update] : updated 2996
[feat] : fixed live-betting withdrawal-system
[feat] : fixed cash-out-feature live-streaming
[fix] : fixed bet-slip-management user-interface
[hotfix] : fixed 3029
[feat] : fixed virtual-reality-experiences advanced-betting-strategies
[feat] : fixed multi-language-support mobile-optimization
[fix] : fixed user-interface-improvements system-stability
[feat] : fixed game-result
[update] : updated 3024
[feat] : fixed promotional-offers social-media-integration
[hotfix] : fixed 3027
[feat] : fixed responsible-gambling statistics-and-analysis
[hotfix] : fixed 3025
[update] : updated 3016
[fix] : fixed odds-calculation user-balance
[fix] : fixed bet-limits fraud-detection
[fix] : fixed bet-slip-errors payment-processing
[fix] : fixed user-verification bet-slip-validation
[feat] : fixed cash-out-feature live-streaming
[feat] : fixed live-betting withdrawal-system
[feat] : fixed loyalty-rewards virtual-reality-betting
[update] : updated 3016
[feat] : fixed artificial-intelligence-predictions social-betting-features
[feat] : fixed mobile-app affiliate-program
[fix] : fixed user-account-security bet-slip-errors
[fix] : fixed bet-cancellation account-management
[hotfix] : fixed 3003
[fix] : fixed user-feedback-system payment-security
[update] : updated 3020
[hotfix] : fixed 2999
[feat] : fixed cash-out-feature live-streaming
[feat] : fixed cash-out-feature live-streaming
[update] : updated 3006
[fix] : fixed data-integrity user-profile
[fix] : fixed bet-slip-management user-interface
[fix] : fixed odds-calculation user-balance
[update] : updated 3004
[hotfix] : fixed 3019
[feat] : fixed promotional-offers social-media-integration
[hotfix] : fixed 3003
[hotfix] : fixed 3039
[update] : updated 3032
[feat] : fixed esports-betting loyalty-program
[fix] : fixed bet-slip-errors payment-processing
[update] : updated 3014
[hotfix] : fixed 3003
[feat] : fixed esports-betting loyalty-program
[update] : updated 3042
[feat] : fixed promotional-offers social-media-integration
[hotfix] : fixed 3039
[hotfix] : fixed 2999
[feat] : fixed multi-language-support mobile-optimization
[fix] : fixed bet-slip-management user-interface
[fix] : fixed match-schedule bet-history
[fix] : fixed data-security bet-settlement
[fix] : fixed data-security bet-settlement
[update] : updated 2996
[fix] : fixed bet-cancellation account-management
[hotfix] : fixed 2997
[hotfix] : fixed 3011
[feat] : fixed cryptocurrency-payments personalized-offers
[fix] : fixed data-integrity user-profile
[fix] : fixed bet-slip-management user-interface
[feat] : fixed cryptocurrency-payments personalized-offers
[hotfix] : fixed 3031
[update] : updated 3034
[hotfix] : fixed 3003
[feat] : fixed referral-system personalized-recommendations
[feat] : fixed promotional-offers social-media-integration
[feat] : fixed blockchain-integration gamification-features
[hotfix] : fixed 3011
[feat] : fixed in-play-betting virtual-sports
[feat] : fixed blockchain-integration gamification-features
[update] : updated 3024
[update] : updated 3020
[fix] : fixed bet-cancellation account-management
[fix] : fixed account-verification deposit-methods
[update] : updated 3020
[hotfix] : fixed 3019
[hotfix] : fixed 3021
[hotfix] : fixed 3013
[update] : updated 3018
[fix] : fixed bet-slip-management user-interface
[feat] : fixed loyalty-rewards virtual-reality-betting
[update] : updated 3036
[feat] : fixed responsible-gambling statistics-and-analysis
[feat] : fixed cryptocurrency-payments personalized-offers
[hotfix] : fixed 3031
[feat] : fixed mobile-app affiliate-program
[feat] : fixed live-betting withdrawal-system
[update] : updated 2994
[fix] : fixed bet-slip-management user-interface
[hotfix] : fixed 3015
[fix] : fixed account-verification deposit-methods
[hotfix] : fixed 2993
[hotfix] : fixed 3021
[hotfix] : fixed 3005
[hotfix] : fixed 3029
[hotfix] : fixed 2995
[fix] : fixed data-integrity user-profile
[update] : updated 3002
[hotfix] : fixed 3027
[update] : updated 3040
[fix] : fixed data-security bet-settlement
[feat] : fixed advanced-betting-options live-dealer-games
[fix] : fixed customer-service user-feedback
[update] : updated 3002
[feat] : fixed promotional-offers social-media-integration
[hotfix] : fixed 3003
[feat] : fixed in-play-betting virtual-sports
[feat] : fixed mobile-app affiliate-program
[fix] : fixed bet-slip-errors payment-processing
[feat] : fixed cryptocurrency-payments personalized-offers
[hotfix] : fixed 3001
[hotfix] : fixed 3029
[hotfix] : fixed 2993
[update] : updated 3034
[update] : updated 3012
[update] : updated 3006
[hotfix] : fixed 3041
[fix] : fixed customer-service user-feedback
[hotfix] : fixed 2999
[update] : updated 3042
[update] : updated 3012
[update] : updated 3002
[fix] : fixed odds-calculation user-balance
[update] : updated 3028
[hotfix] : fixed 3039
[fix] : fixed user-authentication notification-system
[feat] : fixed game-result
[fix] : fixed data-security bet-settlement
[hotfix] : fixed 3033
[update] : updated 3028
[update] : updated 3000
[hotfix] : fixed 3031
[feat] : fixed blockchain-integration gamification-features
[fix] : fixed bet-limits fraud-detection
[feat] : fixed mobile-app affiliate-program
[fix] : fixed odds-calculation user-balance
[hotfix] : fixed 3031
[feat] : fixed esports-betting loyalty-program
[feat] : fixed live-betting withdrawal-system
[hotfix] : fixed 3021
[fix] : fixed data-security bet-settlement
[update] : updated 3002
[fix] : fixed bet-slip-management user-interface
[feat] : fixed promotional-offers social-media-integration
[update] : updated 3020
[hotfix] : fixed 3029
[feat] : fixed in-play-betting virtual-sports
[feat] : fixed virtual-reality-experiences advanced-betting-strategies
[fix] : fixed odds-calculation user-balance
[update] : updated 3034
[hotfix] : fixed 3001
[hotfix] : fixed 3023
[fix] : fixed account-verification deposit-methods
[hotfix] : fixed 3013
[update] : updated 2996
[hotfix] : fixed 3001
t-feature live-streaming
[fix] : fixed system-performance user-experience
[hotfix] : fixed 3021
[fix] : fixed user-feedback-system payment-security
[update] : updated 3010
[fix] : fixed bet-slip-management user-interface
[fix] : fixed account-verification deposit-methods
[update] : updated 3018
[fix] : fixed payment-gateway security-updates
[feat] : fixed responsible-gambling statistics-and-analysis
[fix] : fixed account-verification deposit-methods
[update] : updated 3024
[fix] : fixed bet-cancellation account-management
[hotfix] : fixed 3005
[update] : updated 3020
[feat] : fixed live-betting withdrawal-system
[hotfix] : fixed 3021
[fix] : fixed user-authentication notification-system
[fix] : fixed bet-slip-errors payment-processing
[hotfix] : fixed 2997
[feat] : fixed esports-betting loyalty-program
[hotfix] : fixed 3035
[feat] : fixed in-play-betting virtual-sports
[hotfix] : fixed 2999
[hotfix] : fixed 3041
[update] : updated 3010
[fix] : fixed data-integrity user-profile
[hotfix] : fixed 3013
[update] : updated 3036
[feat] : fixed bonus-system customer-support
[feat] : fixed referral-system personalized-recommendations
[hotfix] : fixed 3021
[hotfix] : fixed 3017
[fix] : fixed account-verification deposit-methods
[update] : updated 3042
[fix] : fixed user-authentication notification-system
[fix] : fixed user-interface-improvements system-stability
[hotfix] : fixed 3003
[feat] : fixed esports-betting loyalty-program
[update] : updated 3006
[feat] : fixed advanced-betting-options live-dealer-games
[update] : updated 3024
[hotfix] : fixed 3019
[fix] : fixed payment-gateway security-updates
[hotfix] : fixed 3011
[feat] : fixed mobile-app affiliate-program
[hotfix] : fixed 3007
[update] : updated 3012
[fix] : fixed bet-cancellation account-management
[hotfix] : fixed 3003
[hotfix] : fixed 2997
[update] : updated 3006
[update] : updated 2994
[update] : updated 3024
[fix] : fixed bet-slip-management user-interface
[update] : updated 2998
[update] : updated 3014
[fix] : fixed data-security bet-settlement
[update] : updated 3020
[feat] : fixed bonus-system customer-support
[update] : updated 3034
[feat] : fixed virtual-reality-experiences advanced-betting-strategies
[fix] : fixed data-security bet-settlement
[update] : updated 3030
[update] : updated 3030
[fix] : fixed bet-cancellation account-management
[hotfix] : fixed 3037
[fix] : fixed account-verification deposit-methods
[fix] : fixed user-interface-updates data-analysis
[update] : updated 3024
[update] : updated 3026
[feat] : fixed tournament-betting live-chat-support
[feat] : fixed artificial-intelligence-predictions social-betting-features
[feat] : fixed in-play-betting virtual-sports
[hotfix] : fixed 3007
[fix] : fixed account-verification deposit-methods
[fix] : fixed payment-gateway security-updates
[hotfix] : fixed 3005
[feat] : fixed in-play-betting virtual-sports
[hotfix] : fixed 3013
[hotfix] : fixed 3013
[update] : updated 2996
[fix] : fixed payment-gateway security-updates
[hotfix] : fixed 3007
[update] : updated 3034
[fix] : fixed user-account-security bet-slip-errors
[update] : updated 3036
[feat] : fixed promotional-offers social-media-integration
[update] : updated 3020
[update] : updated 3042
[feat] : fixed artificial-intelligence-predictions social-betting-features
[feat] : fixed live-betting withdrawal-system
[feat] : fixed live-betting withdrawal-system
[feat] : fixed virtual-reality-experiences advanced-betting-strategies
[fix] : fixed data-integrity user-profile
[fix] : fixed bet-cancellation account-management
[fix] : fixed data-integrity user-profile
[fix] : fixed user-interface-updates data-analysis
[feat] : fixed mobile-app affiliate-program
[hotfix] : fixed 3019
[feat] : fixed mobile-app affiliate-program
[update] : updated 3040
[fix] : fixed payment-gateway security-updates
[feat] : fixed loyalty-rewards virtual-reality-betting
[fix] : fixed odds-calculation user-balance
[update] : updated 2994
[hotfix] : fixed 2999
[update] : updated 3008
[fix] : fixed payment-gateway security-updates
[hotfix] : fixed 3001
[hotfix] : fixed 3033
[update] : updated 3034
[fix] : fixed user-account-security bet-slip-errors
[update] : updated 3014
[fix] : fixed match-schedule bet-history
[fix] : fixed user-interface-improvements system-stability
[feat] : fixed bonus-system customer-support
[fix] : fixed account-verification deposit-methods
