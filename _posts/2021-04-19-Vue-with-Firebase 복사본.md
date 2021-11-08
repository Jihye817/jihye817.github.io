# vue와 firebase 연동하기

## firebase에 웹 연동하기

### 준비 단계
[firebase 콘솔](https://console.firebase.google.com/) 에 들어가면 project를 생성할 수 있다. project를 하나 생성한 다음, `앱에 firebase를 추가하여 시작하기` 라고 쓰여있는 곳 아래의 `</>` 모양으로 생긴 웹 부분을 클릭하여 웹과 연동해준다. firebase sdk를 추가하라고 나오는 코드 부분을 복사하여 붙여넣어주라고 뜨는데, 바로 등록 안 해도 되며 나중에 root에 firebaseConfig.js 파일을 만들어 넣어주면 된다.

### firebase install
프로젝트의 터미널에 다음과 같은 명령어를 입력하여 firebase를 사용할 수 있게 해준다.
````
npm install firebase --save
````

### plugins에 firebase.js 파일 생성

firebase.js 파일에 firebase initialize를 해주어야 한다.
사용할 기능들도 이곳에서 import 해주면 된다.

````js
import Vue from 'vue';
import firebase from 'firebase/app';

//사용할 기능 import
import 'firebase/auth'; //사용자 인증. auth()를 사용할 수 있다.
import 'firebase/firebase-database'; //db

firebase.initializeApp(config 파일 만들어 넣기) //initialize
Vue.prototype.$firebase = firebase //전역으로 사용하기 위한 설정이다.
````

### main.js에 import
만든 firebase 파일을 main.js에서 import 해준다.

````js
import './plugins/firebase';
````

### root에 firebaseConfig.js 파일을 만들어 주기

firebase 프로젝트 설정에 들어가면 `Firebase SDK snippet` 부분에 `구성` 이 있다.
해당 부분을 복사하여 firebaseConfig.js에 붙여넣자.

const firebaseConfig = {...} 으로 하였을 때 오류가 났기 때문에 export default {...} 으로 바꿔 사용하였다.

```
[Vue warn]: Error in v-on handler: "Error: Your API key is invalid, please check you have copied it correctly."
```

### 저장 후 firebase.js에 import 해주기
````js
import Vue from 'vue';
import firebase from 'firebase/app';

import firebaseConfig from '../../firebaseConfig.js'; //위에서 만든 config
//사용할 기능 import
import 'firebase/auth'; //사용자 인증. auth()를 사용할 수 있다.
import 'firebase/firebase-database'; //db

firebase.initializeApp(firebaseConfig) //위에서 만든 config. 여기에 쓰인다.
Vue.prototype.$firebase = firebase //전역으로 사용하기 위한 설정이다.
````
`Vue.prototype.$firebase = firebase` 를 통해 전역에서 사용할 수 있도록 해주었기 때문에, 다른 파일에서도 불러와 사용 가능하다.
`this.$firebase.~`로 사용하면 된다.

---

### 주의사항


> 1

gitignore에 `.firebaserc`, `.firebaseConfig.js`를 등록해주어야 깃허브에 **apiKey**가 올라가는 불상사가 생기지 않는다.

> 2

firebase.auth() 내에서는 this를 사용할 수 없어 `this.$router`에서 오류가 발생한다. (Cannot read property '\$router' of undefined) 이럴 경우 firebase 밖에서 `let self = this;` 와 같이 따로 변수를 정의해준 후 self.$router로 사용하면 해결할 수 있다.

ex)
````js
login() {
  this.$firebase.auth().signInWithEmailAndPassword(this.email, this.password).then(
    function(user){
      this.$router.push("/main"); //this 사용 불가능으로 에러 발생
    },
    function(err) {
      alert("에러 : " + err.message);
    }
  )
}

//
login() {
  let self = this; //변수 self 정의
  this.$firebase.auth().signInWithEmailAndPassword(this.email, this.password).then(
    function(user){
      self.$router.push("/main"); //에러가 발생하지 않는다.
    },
    function(err) {
      alert("에러 : " + err.message);
    }
  )
}
````