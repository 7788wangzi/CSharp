## Yield关键字是按需创
示例中的类型:
```CSharp
    class Person
    {
        public string FirstName { get; set; }

        public string LastName { get; set; }

        public int Age { get; set; }

        public Person(string firstName, string lastName, int age)
        {
            this.FirstName = firstName;
            this.LastName = lastName;
            this.Age = age;
            Console.WriteLine($"Initialized {firstName} {lastName} who is {age}.");
        }        
    }
```

填充数据的两种方式，一种直接使用List来创建集合对象并返回，另一种是使用`yield return`返回`IEnumberable<>`对象。
```CSharp

    class DataAccessor
    {
        public static List<Person> PopulateData()
        {
            List<Person> output = new();

            output.Add(new Person( "Tim", "Speaker", 35 ));
            output.Add(new Person (  "Syndney",  "Noice",  33 ));
            output.Add(new Person (  "Crey",  "Bob",  40 ));

            return output;
        }

        //使用yield按需创建(Crete on-demand)
        public static IEnumerable<Person> PopulateDataYield()
        {
           yield return new Person("Tim", "Speaker", 35);
           yield return new Person("Syndney", "Noice", 33);
           yield return new Person("Crey", "Bob", 40);
        }
    }
```

在客户类中调用，查看两种写法带来的处理顺序的不同
```CSharp
        static void Main(string[] args)
        {
            Console.WriteLine("Start App!");

            var people = DataAccessor.PopulateData();
            foreach (var person in people)
            {
                Console.WriteLine($"Used {person.FirstName} {person.LastName} who is {person.Age}");
            }

            Console.WriteLine("------Yield Sample-------");
            //Use yield
            var people2 = DataAccessor.PopulateDataYield();
            foreach(var person in people2)
            {
                Console.WriteLine($"Used {person.FirstName} {person.LastName} who is {person.Age}");
            }

            Console.WriteLine("Finish App.");
        }
```

程序输出：
```cmdlet
Start App!
Initialized Tim Speaker who is 35.
Initialized Syndney Noice who is 33.
Initialized Crey Bob who is 40.
Used Tim Speaker who is 35
Used Syndney Noice who is 33
Used Crey Bob who is 40
------Yield Sample-------
Initialized Tim Speaker who is 35.
Used Tim Speaker who is 35
Initialized Syndney Noice who is 33.
Used Syndney Noice who is 33
Initialized Crey Bob who is 40.
Used Crey Bob who is 40
Finish App.
```
从程序输出看出，使用yield return的方式，只会在客户端调用的当前item的时候才创建这个item。

IEnumberable<T>对象的另一种调用方式:
```
    var iterater = people2.GetEnumerator();
    while (iterater.MoveNext())
    {
        var person = iterater.Current;
        Console.WriteLine($"Used {person.FirstName} {person.LastName} who is {person.Age}");
    }
```
