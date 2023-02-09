## Function Interface

Para o primeiro exemplo vamos criar um interface normal

```sh
Interface Executor{
  fun execute()
}
```

E vamos usa-la:
```sh
suspend fun scheduleJob( delay: Long ):Executor{
   delay(delay)
   
   return object: Executor {
      override fun executor(){
        println("Executing...")
      }
   }
}
```
Já com o interface de funcao

```sh
fun interface Executor{
  fun execute()
}
```

```sh
suspend fun scheduleJob( delay: Long ):Executor{
   delay(delay)
   
   return  Executor {
        println("Executing...")
   }
}
```
