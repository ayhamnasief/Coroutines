## Coroutines

![image](https://github.com/user-attachments/assets/90e649d4-3257-46cd-bf81-d1c8921bc3c7)

![image (1)](https://github.com/user-attachments/assets/ed4ae6a2-602f-4096-a018-942f700b75da)

![image (2)](https://github.com/user-attachments/assets/e3657d5a-28b5-48a1-8025-b20805e7f851)

#### Run suspend function - Not In the order we want
```java
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        Log.d("ayham", "First")

        GlobalScope.launch(newSingleThreadContext("Ayham Thread")) {
            delayTask("Second")
        }

        Log.d("ayham", "Third")
    }

    suspend fun delayTask(txt:String){
        delay(3000)
        Log.d("ayham", txt+"")
    }
}
```

#### Run suspend function - In the order we want
```java
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        runBlocking { // OR GlobalScope.launch
            Log.d("ayham", "First")
            delayTask("Second")
            Log.d("ayham", "Third")
        }
    }

    suspend fun delayTask(txt:String){
        delay(5000)
        Log.d("ayham", txt+"")
    }
}
```

#### Run two suspend functions in coroutine block ( task2 will wait task1 to finish )
```java
Ex1:
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        GlobalScope.launch {
            delayTask("First")
            delayTask("Second")
        }
    }

    suspend fun delayTask(txt:String){
        delay(5000)
        Log.d("ayham", txt+"")
    }
}

Ex2:
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        runBlocking {
            delayTask("First")
            delayTask("Second")
        }
    }

    suspend fun delayTask(txt:String){
        delay(5000)
        Log.d("ayham", txt+"")
    }
}
```

#### Run two coroutines block in coroutines block
```java
Ex1:
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        runBlocking {
            delayTask("First")
            delayTask("Second")
        }
    }

    suspend fun delayTask(txt:String){
        GlobalScope.launch {
            delay(5000)
            Log.d("ayham", txt+"")
        }
    }
}

Ex2:
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        GlobalScope.launch {
            delayTask("First")
            delayTask("Second")
        }
    }

    suspend fun delayTask(txt:String){
        GlobalScope.launch {
            delay(5000)
            Log.d("ayham", txt+"")
        }
    }
}

Ex3:
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        GlobalScope.launch {
            GlobalScope.launch { delayTask("First") }
            GlobalScope.launch { delayTask("Second") }
        }
    }

    suspend fun delayTask(txt:String){
            delay(5000)
            Log.d("ayham", txt+"")
    }
}
```

#### async AND await
```java
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        GlobalScope.launch {
            val t1 = async { delayTask1() } // Return Deferred value and String inside it
            val t2 = async { delayTask2() } // Return Deferred value and String inside it

            if (t1.await() == t2.await()) {
                Log.d("ayham", "Equals")
            } else {
                Log.d("ayham", "Not Equals")
            }
        }
    }

    suspend fun delayTask1(): String {
        delay(5000)
        return "task"
    }

    suspend fun delayTask2(): String {
        delay(4000)
        return "task"
    }
}
```

#### Jobs & Structured Concurrency

![image (3)](https://github.com/user-attachments/assets/1f73a522-d1fb-4c97-94e1-ac54bfd90479)
```java
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val parentJob: Job = Job()
        GlobalScope.launch {
            val childJob = GlobalScope.launch(parentJob) {
                val child1 = launch { delayTask1() }
                val child2 = launch { delayTask2() }
                launch { delay(2000) }

                child1.join()
                child2.join()
                joinAll(child1, child2)
            }
            parentJob.cancel()
        }
    }

    suspend fun delayTask1(): String {
        delay(5000)
        return "task"
    }

    suspend fun delayTask2(): String {
        delay(4000)
        return "task"
    }
}
```

#### Channels: Deal with hot stream
```java
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        var kotlinChannel:Channel<String> = Channel()
        val charList = arrayOf("A", "B", "C", "D")

        runBlocking {
            launch {
                for(char1 in charList){
                    kotlinChannel.send(char1)
                    delay(3000)
                }
            }

            launch {
                for (char2 in kotlinChannel){
                    Log.d("ayham", char2)
                }
            }
        }
    }
}
```

#### Note:
Problem in channel (Memory leak & in case no collector to consume the hot stream
![image (4)](https://github.com/user-attachments/assets/5968cb5c-4023-40f1-8749-c1c11b95eaf3)

#### Flow: Deal with cold stream
![image (5)](https://github.com/user-attachments/assets/a8990ad8-5af7-43a4-a836-2b9430cc3622)
```java
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        runBlocking {
        // Producer - flow is coroutine
        flow<Int> {
            for (i in 1..10){
                emit(i)
                delay(1000)
                Log.d("ayham", i.toString())
            }
        }.filter { i:Int->i<5 } // Intermediate
            .buffer().collect { // Collector - suspend function that's why we work inside runBlocking
                delay(2000)
                Log.d("ayham", it.toString())
            }
        }
    }
}
```




