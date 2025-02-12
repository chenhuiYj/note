context 是 React 提供的一种直接访问祖先节点上的状态的方法，从而避免了多级组件层层传递 props 的频繁操作。

Context 提供了一个无需为每层组件手动添加 props，就能在组件树间进行数据传递的方法。如果多个组件中都需要某个值，或者获取值和使用值的层级相隔很远，就可以使用 Context 来共享数据。

## 1.语法
createContext‌：创建一个 Context 对象，并可以提供一个默认值。这个默认值只有在组件所处的树中没有匹配到 Provider 时才会生效。
```javascript
const MyContext = React.createContext(defaultValue);
```
‌Provider‌：每个 Context 对象都会返回一个 Provider React 组件。Provider 接收一个 value 属性，这个属性会传递给所有被其包裹的消费组件。
```html
<MyContext.Provider value={someValue}>
  {...}
</MyContext.Provider>
```
‌Consumer‌：用于在组件中读取 Context 的值。接收一个函数作为子元素，这个函数接收当前的 context 值，并返回一个 React 节点。
```html
<MyContext.Consumer>
  {value => /* 在这里可以读取 context value */}
</MyContext.Consumer>
```
useContext‌：在函数组件中，我们可以使用 useContext 钩子来读取 Context 的值。
```javascript
const value = useContext(MyContext);
```
## 2.使用场景

‌主题管理‌：管理和自定义应用程序用户界面的外观，如颜色方案、字体等。使用 Context 可以在整个应用程序中轻松共享相同的主题，而无需手动单独更新每个组件。

‌用户身份验证‌：存储用户信息和身份验证状态，如姓名、电子邮件等，以便显示个性化内容并限制 UI 中某些功能的访问。

‌国际化‌：存储当前的语言设置，以便在应用程序中显示不同语言的文本。

‌全局状态管理‌：如当前用户的设置、应用的配置信息等，这些信息可能在多个组件中都需要使用。
## 3.最佳实践
‌谨慎使用‌：虽然 Context 非常强大，但过度使用会降低组件的复用性。因此，应该只在必要时使用 Context。仅在全局数据上使用 Context，避免过度使用 Context，‌避免在高频更新下使用 Context。

‌组合使用‌：可以将 Context 与其他状态管理工具（如 Redux）结合使用，以更好地管理应用程序的状态。

‌提供默认值‌：在创建 Context 时提供一个有意义的默认值，以避免在没有匹配到 Provider 时出现错误。

‌跨组件通信‌：当某些状态需要在不直接相关的组件之间共享时，可以使用 Context 来避免繁琐的 props 传递。

## 4.举例说明
以下是一个简单的使用 Context 来管理网站语言的例子：

LanguageContext.js
```javascript
import React, { createContext, useState } from 'react';

const language= {
  en:'en',
  tc:'zh_HK',
  sc:'zh_CN'
};

export const LanguageContext= createContext(language.en);

export const LanguageProvider = ({ children }) => {
  const [currentLanguage, setLanguage] = useState(language.en);

  const toggleLanguage = (changeLanguage) => {
    // 切换语言的逻辑
    setLanguage(language[changeLanguage]);
  };

  return (
    <LanguageContext.Provider value={{ currentLanguage, toggleLanguage }}>
      {children}
    </LanguageContext.Provider>
  );
};
```
App.js
```javascript
import React from 'react';
import { LanguageProvider } from './LanguageContext';
import Childfrom './Child';

const App = () => {
  return (
    <LanguageProvider >
      <Child/>
      {/* 其他组件 */}
    </LanguageProvider >
  );
};

export default App;
```
Child.js
```javascript
import React, { useContext } from 'react';
import { LanguageContext} from './LanguageContext';

const btnCopy={
	'en':'button',
	'zh_HK':'按鈕',
	'zh_CN':'按钮'
}

const Child= () => {
  const { currentLanguage, toggleLanguage } = useContext(LanguageContext);

  return (
    <button onClick={toggleTheme(‘sc’)}>
      {btnCopy[currentLanguage]}
    </button>
  );
};

export default Child;
```