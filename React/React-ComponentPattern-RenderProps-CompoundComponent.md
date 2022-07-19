# 【筆記】React Component Pattern - Render Props & Compound Component

###### tags: `筆記文章`

![image alt](https://www.freecodecamp.org/news/content/images/size/w2000/2020/02/Ekran-Resmi-2019-11-18-18.08.13.png)

當我們在開發專案時，一定會碰到要製作『共用 Component』 的時後，而在設計這些共用元件時不免俗的一定會考慮到『可維護性』、『彈性』、『擴充性』等問題，因為在開發過程中常會應各種不同的需求而導致元件的調整，應此常會使用最小程度的擴充來減少改動後對專案的影響。
等等要介紹的 **Render Props** 與 **Compound Component** 是前端元件設計上的一種模式，也是我們較常在專案上會看到或寫到的一種方式，就讓我們往下看看吧！！！

## Render Props

> The term “render prop” refers to a technique for sharing code between React components using a prop whose value is a function.
>
> Render Props 是指一種在 React 组件之間使用 Function 的方式來透過 Prop 共享代碼

**Render Props** 算是 React Component Pattern 中最普遍也是較有名的 Pattern，其實我們在實物上都已經接觸過它，例如：react-router 的 Route 和 React 16.3 後的 Context API，都是 Render Props 的一種型式。

### Render Props 的優點

**Render Props** 的優點在於，能夠讓使用到該元件的地方更方便的操作元件裡的 **_State_**。
例如：『**組件使用者**』能透過 render function 來獲得（子）組件狀態的控制權，則組件使用者就可以用（子）組件的 State 來做程式邏輯上的判斷，或是更改（子）組件的狀態。

### Render Props 的使用

**Render Props 在使用上可以有兩種寫法:**

1. 將 render function 當作 **props** 傳入 Component：
   #### react-router
   ```javascript=
   <Route path='/home' router={({history})=><Component history={history} /> }
   ```
2. 將 render function 當作 **this.props.children** 傳入 Component：
   #### react context API
   ```javascript=
   <Consumer>
       {value => <Component counter={value} />}
   </Consumer>
   ```

### Render Props 實例

#### **需求：透過一個按鈕來顯示開啟或關閉房門的文字訊息**

我們可能第一時間會想到直接寫個 Component 來達成我們的需求（如一）。
但如果我們另一個頁面要**複用此組件**且需要修改『**_顯示文字_**』時，將會碰到無法複用的問題，這時就需要將組件封裝讓元件能被共享（如二）。
但如果不只『顯示文字』需要修改還需依照是否開啟(isOpen)來顯示更多內容時，則就得讓組件**更彈性化**才行（如三）。

**1. 無法共享 Component**

```javascript=
class StateToggle extends React.Component {
  constructor() {
    super();
    this.state = {
      isOpen: false
    };
  }
  handleToggle = () => {
    this.setState(({ isOpen }) => ({ isOpen: !isOpen }));
  };
  render() {
    const { isOpen, title } = this.state;
    return (
      <div>
        // 顯示文字!!!!!!!
        {isOpen ? "The door is open" : "The door is Close"}
      </div>
    );
  }
  // 使用Component
  <StateToggle/>
}
```

**2. 將顯示文字 Message 拉出去當成 Props 傳入給組件**

```javascript=
 render() {
    // Props傳入文字訊息 openMessage,closeMessage
    const { openMessage, closeMessage } = this.props;
    const { isOpen, title } = this.state;
    return (
      <div>
        {isOpen ? openMessage : closeMessage}
      </div>
    );
  }
 // 使用Component
 <StateInPropsToggle
    openMessage={'Open the door~~~'}
    closeMessage={'Close the door~~~'}
 />

```

**3. 透過 Render Props 將子組件的 State 傳出來給組件使用者，組件使用者透過子組件的 isOpen 來判斷並顯示更多資訊**

```javascript=
renderToggle = ({ isOpen, toggle }) => {
    const { title } = this.state;
    return (
      <div>
        <h2 className="title">{title}</h2>
        {isOpen ? this.renderOpenMessage(): this.renderCloseMessage()}
        <hr />
        <button className="btn" onClick={toggle}>
          {isOpen
            ? "click render Props btn Open "
            : "click click render Props btn close"}
        </button>
      </div>
    );
  };
  renderOpenMessage = () =>{
    return(
      <div>
        Opening the door~~~~~~~~~
      </div>
    )
  }
  renderCloseMessage = () =>{
    return(
      <div>
        No~~~~!!! Door is Closing~~~~
      </div>
    )
  }
  // 呼叫 Component
  /* use render function in Props */
  <RenderPropsToggle
    onToggle={isOpen => console.log("toggle", isOpen)}
    renderToggle={this.renderToggle}
  />

  /*  use render function in this.props.children */
  <RenderChildrenToggle
    onToggle={isOpen => console.log("toggle", isOpen)}>
    {this.renderToggle}
  </RenderChildrenToggle>
```

```javascript=
class RenderPropsToggle extends React.Component {
// 部分審略
  render() {
    const { isOpen } = this.state;
    return this.props.renderToggle({
      isOpen: isOpen,
      toggle: this.handleToggle
    });
  }
}
```

```javascript=
class RenderChildrenToggle extends React.Component {
// 部分審略
    render() {
        const { isOpen } = this.state;
        return (
          <div>
            <h2> use this.props.children </h2>
            {this.props.children({ isOpen: isOpen, toggle: this.handleToggle })}
          </div>
        );
    }
}
```

### Render Props 結論

當今天組件變得很複雜時，如果只單單透過 Props 傳入資訊，組件將會擁有過多的 Props 而導致變得肥大、混亂，最後造成難以維護的結局。
這時就可以透過使用 Render Props 的方式，讓組件使用者擁有子組件的狀態來處理更複雜的情況或渲染更多的 DOM 元素，而不是一昧的去擴充組件的 Props。

## Compound Component 複合元件

Compound Component 顧名思義就是將多個組件組成一個組件，並且組件間能夠彼此協同合作，通常會是一個 Component 當作 Parent，Parent 底下有多個 Child Component 來組成 Compound Component。例如 Html 中的下拉選單 <select> 和 <option> 就是一個很好的例子。

```javascript=
<select>
  <option>option1</option>
  <option>option2</option>
  <option>option3</option>
</select>
```

### Compound Component 的優點

Compound Component 的優點在於，能夠將**父組件**的 State 傳遞給**子組件**（this.props.children) 當作子組件的 Props，讓父組件與子組件之間擁有隱含狀態共享（implicit state），而 Compound Component 的設計也方便子組件任意調整順序，且通常會將子組件封裝成父組件中的(static property)來表示兩者之間的關係，例如：React-bootstrap 的 From 組件。

> #### 為何叫作隱含狀態共享（implicit state）⁉
>
> 隱含狀態共享（implicit state）指的是，使用者無需去瞭解父組件與子組件如何溝通、如何傳遞參數，只需將子組件當做 this.props.children 傳入即可，父組件自然會將事先設定好的狀態當作 Props 傳遞給子組件。

### Compound Component 的使用

從優點敘述中可知，在使用 Compound Component 時我們只需要知到**誰是父組件**即可，之後將所需的組件當作 this.props.children 傳入就大功告成了。
**這邊用 React-bootstrap 的 From 組件舉例：**

```javascript=
// React-boostrap Forms
<Form>
    <Form.Label>Email address</Form.Label>
    <Form.Control type="email" placeholder="Enter email" />
    <Form.Text className="text-muted">
      We'll never share your email with anyone else.
    </Form.Text>
</Form>
```

### Compound Component 實例

#### **需求：設計一個『繪畫組件』裡面包含『圓形組件』、『方形組件』且可以依著繪畫組件內的狀態去改變子組件的文字**

我們透過運用 children 的形式來將『繪畫組件』與『圓形組件』、『方形組件』綁在一起。

```javascript=
<DrawCompoundComponent>
    <CircleComponent>方形</CircleComponent>
    <RectComponent>圓形</RectComponent>
</DrawCompoundComponent>
```

但這樣只達成了畫面的部分，我們還需將『繪畫組件』的狀態丟給子組件它們才行，所以將程式碼改成以下。
透過運用 React.Children.map 來將每一個 Child element 取出，並且透過 React.cloneElement 複製一份新的 Child element 並且將『繪畫組件』的狀態(State)與 Child element 原本身上的 Props 進行 shallow merge 後，再當作 Props 傳給新的 Child element。

> **這邊不用 this.props.children.map 而用 React.Children.map 是因為當只有一個 children 時 this.props.children 將返回 object 而不是 array，所以如果使用 this.props.children 則就需要再多做一些判斷。**

```javascript=
class DrawCompoundComponent extends React.Component {
  constructor() {
    super();
    this.state = {
      isClick: false
    };
  }
  renderChildDOM = () => {
    const { children } = this.props;
    let compoundChildren = React.Children.map(children, child => {
      return React.cloneElement(child, {
        ...this.state
      });
    });
    return <div>{compoundChildren}</div>;
  };
  render() {
    return (
      <div>
        {this.renderChildDOM()}
      </div>
    );
  }
}
```

但 React.Children.map 有一個致命性的缺點，就是它只會 map 到第一層的 children，所以下面這種情況時 CompoundRectComponent 則不會被複製到。

```javascript=
<DrawCompoundComponent>
    <CompoundCircleComponent
        width="100px"
        height="100px"
        backgroundColor="#fecdab"
    />
    <div>
        <CompoundRectComponent
            width="100px"
            height="100px"
            backgroundColor="#aebd12"
        />
    </div>
 </DrawCompoundComponent>
```

而 React 16.3 推出的 Context API 正好可以解決這個問題，運用 Provider 與 Consumer 直接將組件內的狀態傳給子組件，而不需再透過 React.Children.map 與 React.cloneElement 的形式來傳遞。

```javascript=
const DrawContext = React.createContext();
class DrawCompoundComponent extends React.Component {
    // ...skip...
    render(){
        return(
            <DrawContext.Provider value={this.state}>
              {this.props.children}
            </DrawContext.Provider>
        )
    }
}
<DrawContext.consumer>
    {contextValue =>(<CompoundRectComponent {...contextValue}/>)}
</DrawContext.consumer>
```

最後我們把『圓形組件』、『方形組件』封裝成 『繪畫組件』的 static property，像一開始所介紹的 React-bootstrap Forms 組件一樣。

> 並不一定需要將組件封裝成父組件的 static property，而封裝的好處在於，組件間較能明確的表達彼此之間的關係（繪圖 -> 圓形、方形...）。

```javascript=
class DrawCompoundComponent extends React.Component {
  static circleContext = props => {
    let contextValue = React.useContext(DrawContext);
    return (
      <CompoundCircleComponent
        width={props.width}
        height={props.height}
        backgroundColor={props.backgroundColor}
        {...props}
        {...contextValue}
      >
        {props.children}
      </CompoundCircleComponent>
    );
  };
  render(){
        return(
            <DrawContext.Provider value={this.state}>
              {this.props.children}
            </DrawContext.Provider>
        )
    }
}
// 組件使用
 <DrawCompoundComponent>
    <DrawCompoundComponent.circleContext
      width="100px"
      height="100px"
      backgroundColor="#fecdab"
    />
 </DrawCompoundComponent>
```

### Compound Component 總結

Compound Component 主要會用在封裝組件上面，像是如果要開源組件到社群就會用到這種 Design Pattern，我們可以運用 Static Property 將同性質或有相互關係的組件通通封裝成一個組件，而我們就可以稱這個組件為 **Compound Component**。

## 總結

今天主要分享了 Render Props & Compound Component，而兩者一個在實務上經常會用 (Render Props)，一個在元件製作與封裝上常會用(Compound Component)，且兩者最大的差異在於：

- Render Props 是將操作權交到**使用該組件的使用者手上**。
- Compound Component 是將操作權交到了**子組件手上**。

最後附上文章內的原始碼～～～
Demo: https://codesandbox.io/s/react-component-pattern-render-props-0z1xv?file=/src/renderProps/RenderPropsToggle.js

## Reference

1. [ArvinH - 進階 React Component Patterns 筆記（上）](https://blog.techbridge.cc/2018/06/27/advanced-react-component-patterns-note/)
1. [Kent C. Dodds - Answers to common questions about render prop](https://kentcdodds.com/blog/answers-to-common-questions-about-render-props)
1. [React 官方 - Render Props](https://reactjs.org/docs/render-props.html)
1. [莫力全 Kyle Mo - Design Pattern In React Component — Compound component (複合元件)](https://medium.com/@oldmo860617/design-pattern-in-react-component-compound-component-%E8%A4%87%E5%90%88%E5%85%83%E4%BB%B6-46ed5fb65459)
1. [Kent C. Dodds - React Hooks: Compound Components](https://kentcdodds.com/blog/compound-components-with-react-hooks)
