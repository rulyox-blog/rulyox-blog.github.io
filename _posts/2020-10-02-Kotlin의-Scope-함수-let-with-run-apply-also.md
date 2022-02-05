---
layout: post
title: Kotlin의 Scope 함수 let, with, run, apply, also
tags: [Kotlin]
comments: true
---

[Scope 함수](https://kotlinlang.org/docs/scope-functions.html)들은 어떤 객체의 context에서 코드를 실행할 수 있도록 해주는 함수들입니다. 이 함수들은 lambda를 이용해 임시 scope를 만들고, 그 scope 안에서는 해당 객체의 이름을 사용하지 않고 접근할 수 있도록 해줍니다.

Scope 함수에 해당하는 `let`, `with`, `run`, `apply`, `also` 5개의 함수들은 비슷한 기능을 갖고 있습니다.

이 함수들 간의 차이점은 `context` 객체에 접근하는 방법과 return 값 입니다.

`run`, `with`, `apply`는 `this`로 객체에 접근하고, `let`, `also`는 `it`으로 객체에 접근합니다.

`apply`, `also는` 객체를 return하고, `let`, `run`, `with`는 lambda 결과를 return합니다.

## let

`let`은 `it`을 통해 객체에 접근할 수 있고, return값은 lambda 결과입니다.

~~~
data class Person(val name: String, val age: Int)

fun main() {

    val person = Person("John", 25)

    val result = person.let {

        println(it.name)
        println(it.age)
        
        "${it.name} is ${it.age} year old"
        
    }

    println(result)

}
~~~

또한, 아래와 같이 safe call 연산자 `?.`를 이용하면 `null`이 아닌 경우에만 lambda 코드를 실행할 수 있도록 할 수 있습니다.

이 방법을 이용하면 `if(person != null) {}` 같은 코드를 대신할 수 있습니다.

~~~
val person: Person? = null

person?.let {
    println(it.name)
}
~~~

## with

`with`는 객체를 파라미터로 받고 `this`를 통해 접근할 수 있습니다. return값은 `let`과 동일하게 lambda 결과입니다.

`with`는 객체를 이용한 함수들을 호출 할 때 유용합니다.

~~~
data class Person(val name: String, val age: Int)

fun main() {

    val person = Person("John", 25)

    with(person) {
        println(this.name)
        println(this.age)
    }

}
~~~

## run

`run`은 `with`와 거의 비슷하지만 확장 함수입니다.

`run`은 lambda 안 에서 객체의 정의와 객체를 사용한 연산을 모두할 때 유용합니다.

~~~
data class Person(var name: String?)

fun main() {

    val person = Person(null)

    person.run {
        name = "John"
        println(this.name)
    }

}
~~~

## apply

`apply`는 `this`로 객체에 접근하고, return 값은 객체 자신입니다. (`this`는 생략해도 됩니다.)

`apply`는 객체의 값들을 설정하는데에 주로 쓰입니다.

~~~
class Person {
    var name: String? = null
    var age: Int? = null
}

fun main() {

    val person = Person().apply {
        name = "John"
        age = 25
    }

    println(person.name)
    println(person.age)

}
~~~

## also

`also`는 `it`으로 객체에 접근하고, return 값은 객체 자신입니다.

`also`는 객체를 이용한 함수들을 호출할 때, 외부 scope의 `this`를 가리지 않고 싶을 때 유용합니다.

~~~
data class Person(val name: String, val age: Int)

fun main() {

    val person = Person("John", 25)

    person.also {
        println(it.name)
        println(it.age)
    }

}
~~~

위 함수들은 비슷한 역할들을 하지만 조금씩 다른 특징이 있기 때문에, 상황에 적합한 함수를 선택해 사용하는 것이 좋습니다.
