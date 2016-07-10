React
=====

*레일즈에서 리액트 작성을 위한 가독성 패턴*

## 목차

1. [Scope](#scope)
1. 구성
  1. [컴포넌트 구성](#component-organization)
  1. [Props 형식화](#formatting-props)
1. 패턴
  1. [계산된 Props](#computed-props)
  1. [혼합 State](#compound-state)
  1. [prefer-ternary-to-sub-render](#prefer-ternary-to-sub-render)
  1. [뷰 컴포넌트](#view-components)
  1. [컨테이너 컴포넌트](#container-components)
1. 안티 패턴
  1. [복합 조건](#compound-conditions)
  1. [render에서 캐시된 State](#cached-state-in-render)
  1. [존재 확인](#existence-checking)
  1. [Props State 설정](#setting-state-from-props)
1. 학습
  1. [핸들러 메쏘드 명칭](#naming-handler-methods)
  1. [이벤트 명칭](#naming-events)
  1. [PropTypes 사용](#using-proptypes)
  1. [엔티티 사용](#using-entities)
1. Gotchas
  1. [테이블](#tables)
1. 라이브러리
  1. [클래스명](#classnames)
1. 그 외
  1. [JSX](#jsx)
  1. [ES2015](#es2015)
  1. [react-rails](#react-rails)
  1. [rails-assets](#rails-assets)
  1. [flux](#flux)

---

## Scope

이것은 레일즈에서 [React.js](https://facebook.github.io/react/) 작성법 입니다.
저희는 적절한 경로를 찾으려 노력했습니다. 여기서 추천하는것들은 좋은 실패한 시도들인 것입니다. 이외에 무언가가 있다면 함께 알도록 찾은것을 공유해주십시오.

모든 예제들은 ES2015 문법으로 작성되어 있습니다. 현재
[공식 react-rails gem](https://github.com/reactjs/react-rails)에는
[babel](http://babeljs.io/)이 탑재되어 있습니다.

**[⬆ 위로](#table-of-contents)**

---

## 컴포넌트 구성

* 클래스 정의
  * 생성자(constructor)
    * 이벤트 핸들러
  * 'component' 생존주기(lifecycle) 이벤트
  * getters
  * render
* 기본 속성(defaultProps)
* 속성 형식(proptypes)

```javascript
class Person extends React.Component {
  constructor (props) {
    super(props);

    this.state = { smiling: false };

    this.handleClick = () => {
      this.setState({smiling: !this.state.smiling});
    };
  }

  componentWillMount () {
    // 이벤트 리스너 추가 (Flux Store, WebSocket, document, 기타.)
  },

  componentDidMount () {
    // React.getDOMNode()
  },

  componentWillUnmount () {
    // 이벤트 리스너 제거 (Flux Store, WebSocket, document, 기타.)
  },

  get smilingMessage () {
    return (this.state.smiling) ? "is smiling" : "";
  }

  render () {
    return (
      <div onClick={this.handleClick}>
        {this.props.name} {this.smilingMessage}
      </div>
    );
  },
}

Person.defaultProps = {
  name: 'Guest'
};

Person.propTypes = {
  name: React.PropTypes.string
};
```

**[⬆ 위로](#table-of-contents)**

## Props 형식화(Formatting)

props가 둘 또는 그 이상이면 줄바꿈 합니다.

```html
// 나쁨
<Person
 firstName="Michael" />

// 좋음
<Person firstName="Michael" />
```

```html
// 나쁨
<Person firstName="Michael" lastName="Chan" occupation="Designer" favoriteFood="Drunken Noodles" />

// 좋음
<Person
 firstName="Michael"
 lastName="Chan"
 occupation="Designer"
 favoriteFood="Drunken Noodles" />
```

**[⬆ 위로](#table-of-contents)**

---

## 계산된 Props

계산된 속성명에는 getters 사용.

```javascript
  // 나쁨
  firstAndLastName () {
    return `${this.props.firstName} ${this.props.lastname}`;
  }

  // 좋음
  get fullName () {
    return `${this.props.firstName} ${this.props.lastname}`;
  }
```

참고: [render에서 캐시된 State](#cached-state-in-render) 안티 패턴

**[⬆ 위로](#table-of-contents)**

---

## 혼합 State

쉬운 파악을 위해 혼합된 state getter 앞에 동사를 붙입니다.

```javascript
// 나쁨
happyAndKnowsIt () {
  return this.state.happy && this.state.knowsIt;
}
```

```javascript
// 좋음
get isHappyAndKnowsIt () {
  return this.state.happy && this.state.knowsIt;
}
```

이런 메쏘드들은 *반드시* `boolean` 값을 반환해야 합니다.

참고: [복합 조건](#compound-conditions) 안티 패턴

**[⬆ 위로](#table-of-contents)**

## 하위 render 에 Ternary 제공

Keep login inside the `render`.

```javascript
// 나쁨
renderSmilingStatement () {
  return <strong>{(this.state.isSmiling) ? " is smiling." : ""}</strong>;
},

render () {
  return <div>{this.props.name}{this.renderSmilingStatement()}</div>;
}
```

```javascript
// 좋음
render () {
  return (
    <div>
      {this.props.name}
      {(this.state.smiling)
        ? <span>is smiling</span>
        : null
      }
    </div>
  );
}
```

**[⬆ 위로](#table-of-contents)**

## 뷰 컴포넌트

컴포넌트는 뷰 안에 작성합시다.
one-off 컴포넌트는 레이아웃과 도메인 컴포넌트들을 병합해버리니 생성하지 마세요.

```javascript
// 나쁨
class PeopleWrappedInBSRow extends React.Component {
  render () {
    return (
      <div className="row">
        <People people={this.state.people} />
      </div>
    );
  }
}
```

```javascript
// 좋음
class BSRow extends React.Component {
  render () {
    return <div className="row">{this.props.children}</div>;
  }
}

class SomeView extends React.Component {
  render () {
    return (
      <BSRow>
        <People people={this.state.people} />
      </BSRow>
    );
  }
}
```

**[⬆ 위로](#table-of-contents)**

## 컨테이너 컴포넌트

> 컨테이너는 데이터를 가져오고(fetching) 그것과 통신하는 하위 컴포넌트를
> render 합니다. 그게 다입니다. &mdash; Jason Bonta

#### 나쁨

```javascript
// CommentList.js

class CommentList extends React.Component {
  getInitialState () {
    return { comments: [] };
  }

  componentDidMount () {
    $.ajax({
      url: "/my-comments.json",
      dataType: 'json',
      success: function(comments) {
        this.setState({comments: comments});
      }.bind(this)
    });
  }

  render () {
    return (
      <ul>
        {this.state.comments.map(({body, author}) => {
          return <li>{body}—{author}</li>;
        })}
      </ul>
    );
  }
}
```

#### 좋음

```javascript
// CommentList.js

class CommentList extends React.Component {
  render() {
    return (
      <ul>
        {this.props.comments.map(({body, author}) => {
          return <li>{body}—{author}</li>;
        })}
      </ul>
    );
  }
}
```

```javascript
// CommentListContainer.js

class CommentListContainer extends React.Component {
  getInitialState () {
    return { comments: [] }
  }

  componentDidMount () {
    $.ajax({
      url: "/my-comments.json",
      dataType: 'json',
      success: function(comments) {
        this.setState({comments: comments});
      }.bind(this)
    });
  }

  render () {
    return <CommentList comments={this.state.comments} />;
  }
}
```

[더읽기](https://medium.com/@learnreact/container-components-c0e67432e005)
[더보기](https://www.youtube.com/watch?v=KYzlpRvWZ6c&t=1351)

**[⬆ 위로](#table-of-contents)**

---

## `render`에서 캐시된 State

`render` 안에 state를 놔두지 마십시오.

```javascript
// 나쁨
render () {
  let name = `Mrs. ${this.props.name}`;

  return <div>{name}</div>;
}

// 좋음
render () {
  return <div>{`Mrs. ${this.props.name}`}</div>;
}
```

```javascript
// 최고
get fancyName () {
  return `Mrs. ${this.props.name}`;
}

render () {
  return <div>{this.fancyName}</div>;
}
```

*This is mostly stylistic and keeps diffs nice. I doubt that there's a significant perf reason to do this.*

참고: [계산된 Props](#computed-props) 패턴

**[⬆ 위로](#table-of-contents)**

## 혼합 조건

`render` 안에 혼합된 조건들을 넣지 마십시오.

```javascript
// 나쁨
render () {
  return <div>{if (this.state.happy && this.state.knowsIt) { return "Clapping hands" }</div>;
}
```

```javascript
// 개선
get isTotesHappy() {
  return this.state.happy && this.state.knowsIt;
},

render() {
  return <div>{(this.isTotesHappy) && "Clapping hands"}</div>;
}
```

The best solution for this would use a [container
component](#container-components) to manage state and
pass new state down as props.

참고: [복합 State](#compound-state) 패턴

**[⬆ 위로](#table-of-contents)**

## 존재 확인

`prop` 객체의 존재 확인 대신 `defaultProps` 사용합시다.

```javascript
// 나쁨
render () {
  if (this.props.person) {
    return <div>{this.props.person.firstName}</div>;
  } else {
    return null;
  }
}
```

```javascript
// 좋음
class MyComponent extends React.Component {
  render() {
    return <div>{this.props.person.firstName}</div>;
  }
}

MyComponent.defaultProps = {
  person: {
    firstName: 'Guest'
  }
};
```

위는 객체나 배열이 사용될때 한정이고,
컴포넌트에서 기대되는 nested 데이터 형식을 확실히 하려면 PropTypes.shape 를 사용합시다.

**[⬆ 위로](#table-of-contents)**

## Props 에서 State 설정

명백한 intent 없이는 props 에서 state를 설정하지 마십시오.

```javascript
// 나쁨
getInitialState () {
  return {
    items: this.props.items
  };
}
```

```javascript
// 좋음
getInitialState () {
  return {
    items: this.props.initialItems
  };
}
```

읽어보기: ["getInitialState 내에서 props는 안티 패턴"](http://facebook.github.io/react/tips/props-in-getInitialState-as-anti-pattern.html)

**[⬆ 위로](#table-of-contents)**

---

## 핸들러 메쏘드 명칭

핸들러 메쏘드 이름은 그것들이 유발된(triggering) 이벤트로 부여.

```javascript
// 나쁨
punchABadger () { /*...*/ },

render () {
  return <div onClick={this.punchABadger} />;
}
```

```javascript
// 좋음
handleClick () { /*...*/ },

render () {
  return <div onClick={this.handleClick} />;
}
```

핸들러 이름은 다음과 같이:

- `handle`로 시작
- 이름의 끝은 그것들이 조종(handle)하는 이벤트로 끝나도록 (예, `Click`, `Change`)
- present-tense 하게

handler들의 애매함을 없애기를 원한다면, `handle`와 이벤트명 사이에 부가 정보를 추가합시다. 예를들어, `onChange` 핸들러들 식별은: `handleNameChange`와 `handleAgeChange`. 이랬을때, 새 컴포넌트를 생성해야 하는지 자신에게 물어보십시오.

**[⬆ 위로](#table-of-contents)**

## 이벤트 명칭

소유받는 이벤트에는 사용자정의 이벤트명을 사용합시다.

```javascript
class Owner extends React.Component {
  handleDelete () {
    // handle Ownee's onDelete event
  }

  render () {
    return <Ownee onDelete={this.handleDelete} />;
  }
}

class Ownee extends React.Component {
  render () {
    return <div onChange={this.props.onDelete} />;
  }
}

Ownee.propTypes = {
  onDelete: React.PropTypes.func.isRequired
};
```

**[⬆ 위로](#table-of-contents)**

## PropTypes 사용

예상되는 통신하기 위해 PropTypes을 사용하고 의미있는 경고를 기록합시다.

```javascript
MyValidatedComponent.propTypes = {
  name: React.PropTypes.string
};
```
`MyValidatedComponent` will log a warning if it receives `name` of a type other than `string`.


```html
<Person name=1337 />
// Warning: Invalid prop `name` of type `number` supplied to `MyValidatedComponent`, expected `string`.
```

컴포넌트들은 `props` 또한 필요할것입니다.

```javascript
MyValidatedComponent.propTypes = {
  name: React.PropTypes.string.isRequired
}
```

이 컴포넌트는 이름의 존재를 검증할것입니다.

```html
<Person />
// 경고: 필수 prop `name`이 `Person`에 특정되지 않음
```

읽기: [Prop Validation](http://facebook.github.io/react/docs/reusable-components.html#prop-validation)

**[⬆ 위로](#table-of-contents)**

## Entities 사용

특수문자들을 위해 React의 `String.fromCharCode()` 사용합시다.

```javascript
// 나쁨
<div>PiCO · Mascot</div>

// 안됨
<div>PiCO &middot; Mascot</div>

// 좋음
<div>{'PiCO ' + String.fromCharCode(183) + ' Mascot'}</div>

// 최고
<div>{`PiCO ${String.fromCharCode(183)} Mascot`}</div>
```

읽기: [JSX Gotchas](http://facebook.github.io/react/docs/jsx-gotchas.html#html-entities)

**[⬆ 위로](#table-of-contents)**

## 테이블

브라우저는 당신을 바보로 여기지만 React는 그렇지 않습니다. `table` 컴포넌트 내에는 항상 `tbody` 사용합시다.

```javascript
// 나쁨
render () {
  return (
    <table>
      <tr>...</tr>
    </table>
  );
}

// 좋음
render () {
  return (
    <table>
      <tbody>
        <tr>...</tr>
      </tbody>
    </table>
  );
}
```

브라우저는 `tbody`를 잊었더라도 삽입해주지만
React는 `table`에 새 `tr`들 삽입을 계속할것이고 혼란을 야기합니다.
항상 `tbody`를 사용하십시오.

**[⬆ 위로](#table-of-contents)**

## 클래스명

조건부 클래스들 관리를 위해 [classNames](https://www.npmjs.com/package/classnames) 사용합시다.

```javascript
// 나쁨
get classes () {
  let classes = ['MyComponent'];

  if (this.state.active) {
    classes.push('MyComponent--active');
  }

  return classes.join(' ');
}

render () {
  return <div className={this.classes} />;
}
```

```javascript
// 좋음
render () {
  let classes = {
    'MyComponent': true,
    'MyComponent--active': this.state.active
  };

  return <div className={classnames(classes)} />;
}
```

읽기: [클래스명 조작](https://github.com/JedWatson/classnames/blob/master/README.md)

**[⬆ 위로](#table-of-contents)**

## JSX

We used to have some hardcore CoffeeScript lovers is the group. The unfortunate
thing about writing templates in CoffeeScript is that it leaves you on the hook
when certain implementations changes that JSX would normally abstract.

We no longer recommend using CoffeeScript to write `render`.

For posterity, you can read about how we used CoffeeScript, when using CoffeeScript was
non-negotiable: [CoffeeScript and JSX](https://slack-files.com/T024L9M0Y-F02HP4JM3-80d714).

**[⬆ 위로](#table-of-contents)**

## ES2015

[react-rails](https://github.com/reactjs/react-rails) now ships with [babel](babeljs.io). Anything
you can do in Babel, you can do in Rails. See the documentation for additional config.

**[⬆ 위로](#table-of-contents)**

## react-rails

[react-rails](https://github.com/reactjs/react-rails) should be used in all
Rails apps that use React. It provides the perfect amount of glue between Rails
conventions and React.

**[⬆ 위로](#table-of-contents)**

## rails-assets
[rails-assets](https://rails-assets.org) should be considered for bundling
js/css assets into your applications. The most popular React-libraries we use
are registered on [Bower](http://bower.io) and can be easily added through
Bundler and react-assets.

**caveats: rails-assets gives you access to bower projects via Sprockets
requires. This is a win for the traditionally hand-wavy approach that Rails
takes with JavaScript. This approach doesn't buy you modularity or the ability to
interop with JS tooling that requires modules.**

**[⬆ 위로](#table-of-contents)**

## flux

flux 구현을 위해 [Alt](http://alt.js.org) 사용합시다. Alt is true to the flux
pattern with the best documentation available.

**[⬆ 위로](#table-of-contents)**
