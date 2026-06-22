# react 业务场景 - 后台管理分步入驻表单（多步骤共享表单数据）

# 一、业务场景（后台百分百高频）

## 1\. 真实业务

后台管理系统【商户入驻审批】功能，拆分3个独立步骤子组件，分步填写、上一步/下一步切换、最终统一提交后端：

- 步骤1：填写商户基础信息（名称、联系人、手机号）

- 步骤2：上传资质资料（营业执照、法人身份证）

- 步骤3：核对全部信息，提交入驻申请

## 2\. 业务痛点

1. 切换上一步、下一步，已填写数据**不能丢失**

2. 三个步骤为独立兄弟子组件，数据需要互通汇总

3. 点击下一步，需要校验当前步骤表单合法性，不通过禁止跳转

4. 最终第三步，拿到三步全部表单数据，统一调用提交接口

## 3\. 技术选型\&实现原理

### 实现方案：状态提升（适配中小型后台，最优最简）

适用：步骤少、层级浅的分步表单

1. 公共父组件：统一管理**全局表单总数据、当前步骤下标**（共享状态）

2. 数据下行：父组件通过props下发表单数据、修改表单的方法

3. 行为上行：每一步子组件修改表单、校验表单，回调通知父组件更新数据

4. 父组件控制步骤切换，校验通过才切换下一步，最终汇总全部数据提交

### 拓展方案（进阶）

步骤超过5步、组件嵌套层级深 → 使用 createContext \+ useContext 跨层共享表单，不用props层层传参。

---

# 二、项目文件结构（企业拆分规范）

```txt
src/
├── Step1Base.jsx   // 步骤1：基础信息子组件
├── Step2File.jsx   // 步骤2：资质上传子组件
├── Step3Submit.jsx // 步骤3：信息确认提交子组件
└── MerchantApply.jsx // 父组件（统一管理状态）
```

---

# 三、全套可运行代码（带详细注释）

## 1\. 父组件：MerchantApply\.jsx（核心：状态提升，统一管控）

```jsx
import { useState } from 'react'
import Step1Base from './Step1Base'
import Step2File from './Step2File'
import Step3Submit from './Step3Submit'

// 父组件：商户入驻总页面，统一共享表单 + 步骤状态
const MerchantApply = () => {
  // ========== 核心共享状态 ==========
  // 当前所在步骤：1/2/3
  const [currentStep, setCurrentStep] = useState(1)
  // 全局汇总表单数据：三个子组件共用，单一数据源
  const [formData, setFormData] = useState({
    // 步骤1数据
    shopName: '',
    contactName: '',
    contactPhone: '',
    // 步骤2数据
    licenseImg: '',
    idCardImg: ''
  })

  // ========== 共享方法：更新全局表单数据 ==========
  // 子组件调用，修改父组件全局表单
  const updateFormData = (newValue) => {
    setFormData({ ...formData, ...newValue })
  }

  // 下一步：校验通过后切换步骤
  const nextStep = () => {
    setCurrentStep(prev => prev + 1)
  }

  // 上一步：直接回退步骤，无需校验
  const prevStep = () => {
    setCurrentStep(prev => prev - 1)
  }

  // 最终提交：后端统一提交全部入驻数据
  const submitAllApply = async () => {
    console.log('✅ 最终提交全部入驻数据：', formData)
    // 此处调用入驻接口：await postMerchantApi(formData)
    alert('入驻申请提交成功！')
  }

  return (
    <div style={{width: '600px',margin: '40px auto',padding: '20px',border: '1px solid #eee',borderRadius: '6px'}}>
      <h2 style={{textAlign:'center'}}>商户入驻审批流程</h2>
      <p style={{textAlign:'center',color:'#1890ff'}}>当前步骤：{currentStep}/3</p>
      <hr/>

      {/* 步骤1组件 */}
      {currentStep === 1 && (
        <Step1Base
          formData={formData}
          updateFormData={updateFormData}
          nextStep={nextStep}
        />
      )}

      {/* 步骤2组件 */}
      {currentStep === 2 && (
        <Step2File
          formData={formData}
          updateFormData={updateFormData}
          nextStep={nextStep}
          prevStep={prevStep}
        />
      )}

      {/* 步骤3组件 */}
      {currentStep === 3 && (
        <Step3Submit
          formData={formData}
          prevStep={prevStep}
          submitAllApply={submitAllApply}
        />
      )}
    </div>
  )
}

export default MerchantApply
```

## 2\. 子组件1：Step1Base\.jsx（步骤1 商户基础信息）

```jsx
import { useState } from 'react'

// 步骤1：填写商户基础信息
const Step1Base = ({ formData, updateFormData, nextStep }) => {
  // 局部校验错误信息
  const [errMsg, setErrMsg] = useState('')

  // 输入框实时修改，同步更新父组件全局表单
  const handleInputChange = (e) => {
    const { name, value } = e.target
    updateFormData({ [name]: value })
  }

  // 下一步校验：非空校验
  const handleNext = () => {
    const { shopName, contactName, contactPhone } = formData
    if(!shopName) return setErrMsg('请输入商户名称')
    if(!contactName) return setErrMsg('请输入联系人')
    if(!contactPhone) return setErrMsg('请输入联系电话')
    // 校验清空，跳转下一步
    setErrMsg('')
    nextStep()
  }

  return (
    <div style={{padding:'10px'}}>
      <h3>第一步：填写商户基础信息</h3>
      <p style={{color:'red',height:'20px'}}>{errMsg}</p>

      <div style={{margin:'12px 0'}}>
        <label>商户名称：</label>
        <input
          name="shopName"
          value={formData.shopName}
          onChange={handleInputChange}
          placeholder="请输入商户名称"
          style={{marginLeft:'8px',padding:'4px'}}
        />
      </div>

      <div style={{margin:'12px 0'}}>
        <label>联系人：</label>
        <input
          name="contactName"
          value={formData.contactName}
          onChange={handleInputChange}
          placeholder="请输入联系人"
          style={{marginLeft:'8px',padding:'4px'}}
        />
      </div>

      <div style={{margin:'12px 0'}}>
        <label>联系电话：</label>
        <input
          name="contactPhone"
          value={formData.contactPhone}
          onChange={handleInputChange}
          placeholder="请输入手机号"
          style={{marginLeft:'8px',padding:'4px'}}
        />
      </div>

      <button onClick={handleNext} style={{marginTop:'20px',padding:'6px 16px',background:'#1890ff',color:'#fff',border:'none',borderRadius:'4px'}}>
        下一步
      </button>
    </div>
  )
}

export default Step1Base
```

## 3\. 子组件2：Step2File\.jsx（步骤2 资质上传）

```jsx
import { useState } from 'react'

// 步骤2：资质图片上传（模拟上传）
const Step2File = ({ formData, updateFormData, nextStep, prevStep }) => {
  const [errMsg, setErrMsg] = useState('')

  // 模拟图片上传，赋值图片地址
  const handleUpload = (key) => {
    // 模拟后端返回图片线上地址
    const mockImgUrl = 'https://mock-img.com/123.png'
    updateFormData({ [key]: mockImgUrl })
  }

  // 下一步校验：必须上传两张资质图
  const handleNext = () => {
    const { licenseImg, idCardImg } = formData
    if(!licenseImg) return setErrMsg('请上传营业执照')
    if(!idCardImg) return setErrMsg('请上传法人身份证')
    setErrMsg('')
    nextStep()
  }

  return (
    <div style={{padding:'10px'}}>
      <h3>第二步：上传入驻资质</h3>
      <p style={{color:'red',height:'20px'}}>{errMsg}</p>

      <div style={{margin:'15px 0'}}>
        <p>营业执照：{formData.licenseImg ? '✅已上传' : '❌未上传'}</p>
        <button onClick={()=>handleUpload('licenseImg')}>模拟上传执照</button>
      </div>

      <div style={{margin:'15px 0'}}>
        <p>法人身份证：{formData.idCardImg ? '✅已上传' : '❌未上传'}</p>
        <button onClick={()=>handleUpload('idCardImg')}>模拟上传身份证</button>
      </div>

      <button onClick={prevStep} style={{marginRight:'10px',padding:'6px 16px'}}>上一步</button>
      <button onClick={handleNext} style={{padding:'6px 16px',background:'#1890ff',color:'#fff',border:'none',borderRadius:'4px'}}>下一步</button>
    </div>
  )
}

export default Step2File
```

## 4\. 子组件3：Step3Submit\.jsx（步骤3 信息确认\+提交）

```jsx
// 步骤3：全部信息核对 + 最终提交
const Step3Submit = ({ formData, prevStep, submitAllApply }) => {
  const { shopName, contactName, contactPhone, licenseImg, idCardImg } = formData
  return (
    <div style={{padding:'10px'}}>
      <h3>第三步：核对入驻信息并提交</h3>
      <div style={{border:'1px solid #f5f5f5',padding:'16px',borderRadius:'4px'}}>
        <p>商户名称：{shopName}</p>
        <p>对接联系人：{contactName}</p>
        <p>联系手机号：{contactPhone}</p>
        <p>营业执照：{licenseImg ? '已上传' : '未上传'}</p>
        <p>法人身份证：{idCardImg ? '已上传' : '未上传'}</p>
      </div>

      <button onClick={prevStep} style={{margin:'20px 10px 0 0',padding:'6px 16px'}}>上一步</button>
      <button onClick={submitAllApply} style={{marginTop:'20px',padding:'6px 16px',background:'#00b42a',color:'#fff',border:'none',borderRadius:'4px'}}>提交入驻申请</button>
    </div>
  )
}

export default Step3Submit
```

---

# 四、完整执行流程（面试口述必背）

1. 父组件提升全局共享状态：`currentStep`步骤下标、`formData`全量表单数据

2. 三个步骤子组件为同级兄弟组件，接收父组件props：表单数据、修改方法、步骤切换方法

3. 子组件输入/上传操作，调用父组件`updateFormData`，修改父组件全局表单

4. 点击下一步：子组件先做局部表单校验，校验通过调用父组件nextStep切换步骤

5. 切换上一步：直接回退步骤，保留所有已填写数据，不会清空

6. 第三步页面，读取父组件完整formData，核对信息后统一提交后端接口

---

# 五、面试官高频追问\+标准答案

## 追问1：为什么分步表单数据，不能每个步骤单独存useState？

**标准答案：**

1. 各步骤为兄弟组件，单独维护state数据割裂，无法汇总提交

2. 上一步回退、下一步前进，页面重渲染会导致填写数据丢失

3. 统一提升至父组件管理，保证**单一数据源**，数据统一、好校验、好提交

## 追问2：什么时候用状态提升、什么时候用Context做分步表单？

**标准答案：**

- **状态提升**：步骤3\-4个以内、组件层级浅、后台简单入驻表单，成本最低，代码简单

- **Context跨层共享**：步骤多、组件嵌套3层以上，避免props逐层透传，简化代码

## 追问3：切换步骤，如何防止数据丢失？

**标准答案：**

不销毁表单数据，只通过**条件渲染\{currentStep===1\}**控制组件显示隐藏，全局formData始终保留，页面不销毁state，数据永久缓存。

## 追问4：大型项目分步表单，还能怎么优化？

1. 使用useContext封装全局表单上下文，消除props传参

2. 结合localStorage临时缓存表单，刷新页面不丢失

3. 使用react\-hook\-form统一做分步表单校验，简化校验代码

4. 复杂跨页面分步，使用Zustand全局库持久化表单数据

## 追问5：下一步校验逻辑，放子组件还是父组件？

**标准答案：**

简单校验放子组件（解耦）；全局联动校验、跨步骤关联校验，统一放到父组件集中管控。
