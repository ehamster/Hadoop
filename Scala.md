完全的面向对象语言
================
对象，类，方法，字段
import java.awt.Color  // 引入Color
 
import java.awt._  // 引入包内所有成员
 
def handler(evt: event.ActionEvent) { // java.awt.event.ActionEvent
  ...  // 因为引入了java.awt，所以可以省去前面的部分
}
import java.awt.{Color, Font}
 
// 重命名成员
import java.util.{HashMap => JavaHashMap}
 
// 隐藏成员
import java.util.{HashMap => _, _} // 引入了util包的所有成员，但是HashMap被隐藏了



变量： var myVar : String = "Too"
常量:  val m : Int = 3

在 scala 中，对保护（Protected）成员的访问比 java 更严格一些。因为它只允许保护成员在定义了该成员的的类的子类中被访问。
而在java中，用protected关键字修饰的成员，除了定义了该成员的类的子类可以访问，同一个包里的其他类也可以进行访问。

作用域
private[x] 

或 

protected[x]
这里的x指代某个所属的包、类或单例对象。
如果写成private[x],读作"这个成员除了对[…]中的类或[…]中的包中的类及它们的伴生对像可见外，对其它所有类都是private。

IF Else = Java
while do while = java
for:
       for( a <- 1 to 10){} //包括10
       for( a <- 1 until 10){} // 不包括10
        
     for( a <- 1 to 3; b <- 1 to 3){} // 迭代9次
   
 过滤循环：
   for( var x <- List
      if condition1; if condition2...
   ){
   statement(s);
   
   Yield：
     var retVal = for{ var x <- List
     if condition1; if condition2...
}yield x

object Test {
   def main(args: Array[String]) {
      var a = 0;
      val numList = List(1,2,3,4,5,6,7,8,9,10);

      // for 循环
      var retVal = for{ a <- numList 
                        if a != 3; if a < 8
                      }yield a

      // 输出返回值
      for( a <- retVal){
         println( "Value of a: " + a );
      }
   }
}


在类中定义的函数就是方法
----------------

class Test{
  def m(x: Int) = x + 3    
  val f = (x: Int) => x + 3  //函数
}
 def addInt( a:Int, b:Int ) : Int = {    return sum}
 def addInt( a:Int, b:Int ) : Unit = { }  //Unit类似于 volid  
  

闭包：
---------------

包通常来讲可以简单的认为是可以访问一个函数里面局部变量的另外一个函数。
object Test {  
   def main(args: Array[String]) {  
      println( "muliplier(1) value = " +  multiplier(1) )  
      println( "muliplier(2) value = " +  multiplier(2) )  
   }  
   var factor = 3  
   val multiplier = (i:Int) => i * factor  
}  

String就是java里面的

数组：
 var myList = Array(1.9, 2.9, 3.4, 3.5)
      
      // 输出所有数组元素
      for ( x <- myList ) {
         println( x )
      }
      
    注意使用 myList(1)  而不是myList[1]
    
   多维数组：
   import Array._  

    var myMatrix = ofDim[Int](3,3)
  
    for (i <- 0 to 2) {
        for (j <- 0 to 2) {
            myMatrix(i)(j) = j;
        }
    }
      
   合并数组：
     var myList1 = Array(1.9, 2.9, 3.4, 3.5)
      var myList2 = Array(8.9, 7.9, 0.4, 1.5)

      var myList3 =  concat( myList1, myList2)
      
      
        var myList1 = range(10, 20, 2)//步长为2
        
        
Collection
    1.List  不可变
    val dim: List[List[Int]] =
   List(
      List(1, 0, 0),
      List(0, 1, 0),
      List(0, 0, 1)
   )
   
   2.Set默认调用的是不可变的，如果想要用可变的需要使用import scala.collection.mutable.Set // 可以在任何地方引入 可变集合
   
   val mutableSet = Set(1,2,3)
println(mutableSet.getClass.getName) // scala.collection.mutable.HashSet

mutableSet.add(4)
mutableSet.remove(1)
mutableSet += 5
mutableSet -= 2

匿名函数,因为在scala中函数可以作argument：
    val x = t.filter(argument => function body)
    因为filter的参数要求是 (Int => Boolean)
    所以可以这么写
    val x = t.filter(e => e%2 == 0)  #t是DataSet
    
scala读取jar包外配置文件的方法  
val directory = new File("")
val filePath = directory.getAbsolutePath
val postgprop = new Properties()

val ipstream = new BufferedInputSteram(new FileInputStream(filePath + "conf/config.properties")
postgprop.load(ipstream)

在工程文件同目录新建conf/config.properties
内容就是
Name=value



这样就能得到value
postgprop.getProperty("Name")
    
