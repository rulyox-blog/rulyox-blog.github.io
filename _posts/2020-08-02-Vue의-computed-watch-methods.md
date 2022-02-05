---
layout: post
title: Vue의 computed, watch, methods
tags: [Vue.js]
comments: true
---

`computed`, `watch`, `methods` 는 Vue 인스턴스가 가질 수 있는 속성들입니다. 이 3가지 속성은 Vue의 중요한 특징이자 자주 쓰이는 기능들이기 때문에 정리했습니다.

## computed

Vue 인스턴스의 data 속성은 해당 인스턴스에서 사용할 수 있는 변수들이고 항상 data binding 되어 있습니다.

`computed`도 data binding 되어 있는 변수라는 점에서 공통점이 있지만, `computed`는 항상 up-to-date 하다는 특징이 있습니다.

~~~
<div id="app">
    <p>{{message}}</p>
    <p>{{reversedMessage}}</p>
</div>
        
<script>
new Vue({
    el: '#app',
    data: {
        message: 'This is message.'
    },
    computed: {

        reversedMessage: function() {
            return this.message.split('').reverse().join('');
        }

    }
});
</script>
~~~

위의 코드를 보면, `computed` 속성 안에서 `reversedMessage`를 함수로써 정의했습니다. 그리고 `reversedMessage`는 `this.message`를 뒤집은 값을 반환하고 있습니다.

이렇게 정의된 `computed` 변수는 이 인스턴스에서 data 변수와 동일하게 사용할 수 있지만, `this.message`의 값이 변경된다면, 항상 그에 맞춰 업데이트된다는 특징이 있습니다.

## watch

`watch` 속성은 어떤 변수의 값이 변경되었을 때 자동으로 호출할 함수를 정의할 때 사용됩니다.

~~~
<div id="app">
    <input v-model="message">
</div>
        
<script>
new Vue({
    el: '#app',
    data: {
        message: 'This is message.'
    },
    watch: {

        message: function(data) {
            console.log(data);
        }

    }
});
</script>
~~~

위 예시에서는 `input`에 data의 `message`가 바인딩되어 있습니다. 따라서 사용자가 `input`을 수정하면 `message`값도 업데이트 됩니다.

이때, `watch` 속성에서 `message`에 어떤 함수를 정의했습니다. 이 함수는 data의 `message`값이 변경될 때 마다 호출됩니다. 이 함수의 파라미터인 data는 `message`의 값 입니다. 따라서, 사용자가 `input`을 수정할 때 마다 `console.log(data)`가 실행됩니다.

## methods

`methods`는 해당 Vue 인스턴스에서 사용할 함수들을 정의하는 곳 입니다.

~~~
<div id="app">
    <p>{{message}}</p>
    <button v-on:click="reverseMsg">Reverse Message</button>
</div>
        
<script>
new Vue({
    el: '#app',
    data: {
        message: 'This is message.'
    },
    methods: {

        reverseMsg: function() {
            this.message = this.message.split('').reverse().join('');
        }

    }
});
</script>
~~~

위 코드에서는 `reverseMsg`라는 함수를 정의했습니다. 이 함수는 `this.message`의 값을 반전시키고 다시 저장하는 함수입니다. DOM 요소인 button을 누르면 `reverseMsg` 함수를 호출하도록 했기 때문에, button을 누를 때 마다 `message` 값이 바뀌는 것을 확인할 수 있습니다.
