#  Dicas
## _Usando o inline e o reified_


 Reified é um tipo especial de palavra-chave que
 ajuda os desenvolvedores Kotlin a acessar as informações
 relacionadas a uma classe em tempo de execução.
 Reified só pode ser usado com funções INLINE .
 Quando a palavra-chave Reified é usada, o compilador copia
 o bytecode da função para cada seção do código onde a função
 foi chamada. Desta forma, o tipo genérico T será atribuído ao
 tipo do valor que recebe como argumento.

## 1 Example - Erro em run time
```sh
fun<T> String.toKotlinJson(): T {
    val mapper = jacksonObjectMapper()
    //Cannot use 'T' as reified type parameter. Use a class instead.
    //ERROR: here: T::class.java
    return mapper.readValue(this, T::class.java)
}
```
## 2 Example - jeito aceitavel
```sh
fun <T : Any> String.toKotlinJson(c: KClass<T>): T {
    val mapper = jacksonObjectMapper()
    return mapper.readValue(this, c.java)
}
```

## 3 Example - jeito KOTLIN
```sh
inline fun <reified T : Any> String.toKotlinJsonWay(): T {
    val mapper = jacksonObjectMapper()
    return mapper.readValue(this, T::class.java)
}
```

## 4 Example - mais exemplos
```sh
inline fun <reified T> myExample(name: T) {
    println("Name of your website -> "+name)
    println("Type of myClass: ${T::class.java}")
}
```



# Como usar:
```sh
data class MyJson(val name: String)
val myJson = """ {"name" :  "myJson"}"""

main(){
  //uso do exampe 2
  myJson.toKotlinJson(MyJson::class)    
  
  //uso examploe 3
  myJson.toKotlinJsonWay<MyJson>()
  
  //uso example 4
    myExample("Paulo Oliveira") //String
    myExample(100)  //Int
    myExample(1L)  //Long
}
```
