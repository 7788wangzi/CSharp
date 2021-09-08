## 如何使用泛型来减少重复代码举例

在从本地文件中保存和读取数据的场景中，如果我们有多个对象类型，则需要为每一个对象分别编写`Save`和`Read`方法， 传统实现是这样子的。

有个类型Person的定义如下：
```CSharp
    class Person
    {
        public string FirstName { get; set; }

        public string LastName { get; set; }

        public int Age { get; set; }
        
    }
```

初始化列表的方法：
```CSharp
    class DataAccessor
    {
        public static List<Person> PopulateData()
        {
            List<Person> output = new();

            output.Add(new Person { FirstName="Tim", LastName="Speaker", Age=35 });
            output.Add(new Person { FirstName = "Sydney", LastName = "Noice", Age = 33 });
            output.Add(new Person { FirstName = "Crey", LastName = "Bob", Age = 40 });

            return output;
        }
    }
```

### 传统实现

在客户类调用的时候，针对`Person`对象的类型， 我们需要调用`SaveToFile(List<Person> People, string filePath)`和`List<Person> ReadFromFile(string filePath)`方法， 假使我们还有其他对象类型也需要`Save`和`Read`操作，那就需要重新编写新类型的`Save`和`Read`方法。例如， `SaveToFile(List<Log> logs, string filePath)`和`List<Log> ReadFromFile(string filePath)`.
```CSharp
      static void Main(string[] args)
        {
            Console.WriteLine("Start App!");
            string csvFile = @"data.csv";

            var people = DataAccessor.PopulateData();

            OperateLocalFile.SaveToFile(people, csvFile);
            var readPeopleList = OperateLocalFile.ReadFromFile(csvFile);

            foreach (var person in readPeopleList)
            {
                Console.WriteLine($"{person.FirstName} {person.LastName} is {person.Age}");
            }

            Console.WriteLine("Finish App.");
        }
```

`OperateLocalFile`的两个`Save`和`Read`方法代码：
```CSharp
        public static void SaveToFile(List<Person> people, string filePath)
        {
            List<string> lines = new();
            foreach (var person in people)
            {
                lines.Add($"{ person.FirstName }, { person.LastName }, { person.Age }");
            }

            File.WriteAllLines(filePath, lines);
        }

        public static List<Person> ReadFromFile(string filePath)
        {
            var lines = File.ReadAllLines(filePath);

            List<Person> people = new List<Person>();

            foreach (var line in lines)
            {
                string[] colums = line.Split(',');
                if (colums.Length != 3)
                    continue;

                int age = 0;
                Int32.TryParse(colums[2], out age);
                people.Add(new Person { FirstName = colums[0], LastName = colums[1], Age = age });
            }
            return people;
        }
```

### 泛型实现
有了泛型以后，我们就可以写一个通用的`Save`和`Read`方法， 在客户类调用的时候再将具体的类型传递到通用方法中。通用方法的签名例如`SaveToFile<T>(List<T> data, string filePath)`和`List<T> ReadFromFile(string filePath)`.
```CSharp
        static void Main(string[] args)
        {
            Console.WriteLine("Start App!");
            string csvFile = @"data.csv";

            var people = DataAccessor.PopulateData();

            //OperateLocalFile.SaveToFile(people, csvFile);
            //var readPeopleList = OperateLocalFile.ReadFromFile(csvFile);

            //使用泛型，减少重复代码（如果不用泛型，需要为每个类型编写Save和Read方法，使用泛型以后，可以用一套方法实现所有类型的Save和Read）
            OperateLocalFile.SaveToFile<Person>(people, csvFile);
            var readPeopleList = OperateLocalFile.ReadFromFile<Person>(csvFile);


            foreach (var person in readPeopleList)
            {
                Console.WriteLine($"{person.FirstName} {person.LastName} is {person.Age}");
            }

            Console.WriteLine("Finish App.");
        }
```

`OperateLocalFile`的两个`SaveToFile<T>(List<T> data, string filePath)`和`List<T> ReadFromFile<T>(string filePath)`方法代码：
```CSharp
   class OperateLocalFile
    {

        public static void SaveToFile<T>(List<T> data, string filePath) where T: class
        {
            List<string> lines = new();
            System.Text.StringBuilder line = new System.Text.StringBuilder();

            var cols = data[0].GetType().GetProperties();

            //save header
            foreach(var col in cols)
            {
                line.Append(col.Name);
                line.Append(",");
            }

            lines.Add(line.ToString().TrimEnd(','));

            //save data
            foreach(var row in data)
            {
                line = new System.Text.StringBuilder();
                foreach (var col in cols)
                {
                    line.Append(col.GetValue(row));
                    line.Append(",");
                }


                lines.Add(line.ToString().TrimEnd(','));
            }

            File.WriteAllLines(filePath, lines);
        }

        public static List<T> ReadFromFile<T>(string filePath) where T: class, new()
        {
            List<T> output = new();

            T entry = new T();
            var cols = entry.GetType().GetProperties();
            var lines = File.ReadAllLines(filePath);

            if(lines == null || lines.Length <2)
            {
                Console.WriteLine("Invalid data");
                return null;
            }

            //read headers
            var headers = lines[0].Split(',');
            

            foreach(var line in lines)
            {
                
                string[] values = line.Split(',');

                //ignore header
                if (values[0] == headers[0])
                    continue;

                entry = new T();
                for (int i = 0; i< headers.Length; i++)
                {
                    foreach (var col in cols)
                    {
                        if (col.Name == headers[i])
                        {
                            col.SetValue(entry, Convert.ChangeType(values[i], col.PropertyType));
                        }
                    }
                }                

                output.Add(entry);
            }

            return output;
        }
    }
```




