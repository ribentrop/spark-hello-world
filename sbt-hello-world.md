
Видюшку найти

## Play with Scala online
https://scalafiddle.io/
https://scastie.scala-lang.org/

## Play with Sbt 
Здесь описано создане простого Scala проекта  c использованием Scala building tool. Spark пока не трогаем, будем иметь дело только со Scala. Если не хочется возиться с ручной подготовкой проекта и сразу хочется Spark, то можно перейти к п.ХХХ
#### Источники
Что такое sbt и зачем он нужен, можно почитать вот тут: ХХХ
--- ???--- Suppose we wish to write a self-contained application using the Spark API. We will walk through a simple application in Scala (with sbt), Java (with Maven), and Python (pip).

Инструкция по установке есть адаптированный мануал отсюда:
https://www.scala-sbt.org/1.x/docs/Installing-sbt-on-Linux.html

Все манипуляции с проектом есть адаптированный маиуал отсюда:
https://www.scala-sbt.org/1.x/docs/sbt-by-example.html

#### Что будем делать:
ХХХ
Цели ХХХ:
- представление о стурктуре приложения|проекта Scala
- зависимости и тп.
- понимать структуту папок (быть способным создать проект Spark самостоятельно без sbt)
- зачем нужны тесты
#### Окружение
Все работы велись на Centos:
```sh
[justribentrop_cloud@sbt-scala-spark-2 ~]$ sudo cat /etc/centos-release
CentOS Linux release 7.7.1908 (Core)
```
Все действия с sbt должны работать и на Windows, но у меня не завелось. Пробовать не рекомендую.
### Sbt Example
Let’s start with examples rather than explaining how sbt works or why.
#### Update repos
```sh
sudo yum update -y
```
#### Install JDK 
You must first install a JDK. We recommend AdoptOpenJDK JDK 8 or JDK 11.
```sh
sudo yum install java-1.8.0-openjdk-devel
```
#### Installing sbt 
```sh
curl https://bintray.com/sbt/rpm/rpm | sudo tee /etc/yum.repos.d/bintray-sbt-rpm.repo
sudo yum install sbt -y
```
#### Create a minimum sbt build 
Далее будем создавать проект в "полуавтоматическом" режиме
Создадим главную папку проекта и основной конфигурационный файл проекта - build.sbt
```sh
$ mkdir foo-build
$ cd foo-build
$ touch build.sbt
```
#### Start sbt shell 
```sh
$ sbt
[info] Updated file /tmp/foo-build/project/build.properties: set sbt.version to 1.1.4
[info] Loading project definition from /tmp/foo-build/project
[info] Loading settings from build.sbt ...
[info] Set current project to foo-build (in build file:/tmp/foo-build/)
[info] sbt server started at local:///Users/eed3si9n/.sbt/1.0/server/abc4fb6c89985a00fd95/sock
sbt:foo-build>
```
При этом sbt подтянет дефорлные зависимости для проекта и создаст структуру папок дефолтного проекта. Можно создавать проекты по шаблонам (например, проект спецмально дляисполнеиня в Spark), но об этом позже.

Про структуру папок проекта можно почитать тут:
https://medium.com/@wangyunlongau/scala-sbt-project-directory-structure-c254bb08623e
https://www.scala-sbt.org/1.x/docs/Directories.html
Можно сделать  tree и посмотреть структуру папок глазами:
```sh
$ sudo yum install tree -y
$ tree
```

Забегая вперед - создавая Scala приложение без sbt можно создать структуру папок самостоятельно ручками (https://medium.com/luckspark/scala-spark-tutorial-1-hello-world-7e66747faec). 
Sbt делает всю рутину за нас, на то он и инстумент для автоматизации построения приложений. Рутина в нашем случае это - создание папок, разрешение зависимостей, подкачка библиотек и т.п.
#### Exit sbt shell 
To leave sbt shell, type exit or use Ctrl+D (Unix) or Ctrl+Z (Windows).
```sh
sbt:foo-build> exit
```
#### Compile a project  
Запустим sbt и скомпилируем пустой проект
```sh
$ sbt
sbt:foo-build> compile
```
Подробнее. что есть compile и прочие команды sbt, которые еще будут впереди (compile,package,run,test,assembly), можно почитать тут:
https://alvinalexander.com/scala/sbt-how-to-compile-run-package-scala-project
Компиляция это компиляция, можно запускать скомпилированное, но исполняемого файла не создается.
Т.к. в нашем случае нет исходников, то compile не создаст ничего такого, что можно было бы запустить.
#### Create a source file 
From a  shell or in your file manager create in the foo-build directory the following nested directories: src/main/scala/example. Then, create Hello.scala in the example directory using your favorite editor as follows:
```sh
package example

object Hello extends App {
  println("Hello")
}
```
#### Recompile on code change  
Run 'compile' command again:
```sh
$ sbt
sbt:foo-build> compile
[info] Compiling 1 Scala source to /tmp/foo-build/target/scala-2.12/classes ...
[info] Done compiling.
[success] Total time: 2 s, completed May 6, 2018 3:53:42 PM
```
Sbt создал (как минимум) скомпилированные классы в /home/justribentrop_cloud/foo-build/target/scala-2.12/classes/example. Это бинарники, больше ничего о них не знаю.
#### Run a previous command 
From sbt shell, press up-arrow twice to find the compile command that you executed at the beginning.
#### Run your app 
```sh
sbt:foo-build> run
[info] Packaging /tmp/foo-build/target/scala-2.12/foo-build_2.12-0.1.0-SNAPSHOT.jar ...
[info] Done packaging.
[info] Running example.Hello
Hello
[success] Total time: 1 s, completed May 6, 2018 4:10:44 PM
```
Что происходит при запуске - ?

# ------------------------------------
# продолжить отсюда
# ------------------------------------


#### Set ThisBuild / scalaVersion from sbt shell 
```sh
sbt:foo-build> set ThisBuild / scalaVersion := "2.12.7"
[info] Defining ThisBuild / scalaVersion
```
#### Check the scalaVersion setting:
```sh
sbt:foo-build> scalaVersion
[info] 2.12.10
```
#### Save/load session
В командной строке sbt можно определять версии spark/scala для своего проекта, подключать зависимости. Все выполеннное в команднойстроке sbt можно сохранить как результат работы сессии в основном настроечном файле проекта build.sbt 
We can save the ad-hoc settings using session save.
```sh
sbt:foo-build> session save
[info] Reapplying settings...
```
При созранении сесии все текущие настройки окружения, выполненные из командной строки sbt> сохраняются в build.sbt


#### Package - ???



#### Save the session to build.sbt 
We can save the ad-hoc settings using session save.
Что происходит при сохранении сессии - ?
```sh
sbt:foo-build> session save
[info] Reapplying settings...
```
build.sbt file should now contain:
```sh
ThisBuild / scalaVersion := "2.12.10"
```
#### Name your project 
Using an editor, change build.sbt as follows:
```sh
ThisBuild / scalaVersion := "2.12.10"
ThisBuild / organization := "com.example"

lazy val hello = (project in file("."))
  .settings(
    name := "Hello"
  )
```
#### Reload the build 
Use the "reload" command to reload the build. The command causes the build.sbt file to be re-read, and its settings applied.
```sh
sbt:foo-build> reload
[info] Loading project definition from /tmp/foo-build/project
[info] Loading settings from build.sbt ...
[info] Set current project to Hello (in build file:/tmp/foo-build/)
sbt:Hello>
```
Note that the prompt has now changed to sbt:Hello>.
#### Run tests
```sh
sbt:Hello> test
[info] Updating 
...
[info] Run completed in 87 milliseconds.
[info] Total number of tests run: 0
[info] Suites: completed 0, aborted 0
[info] Tests: succeeded 0, failed 0, canceled 0, ignored 0, pending 0
[info] No tests were executed.
[success] Total time: 7 s, completed Jan 9, 2020 11:14:32 AM
```





#### Write a test
Leaving the previous command running, create a file named src/test/scala/HelloSpec.scala using an editor:
```sh
import org.scalatest._

class HelloSpec extends FunSuite with DiagrammedAssertions {
  test("Hello should start with H") {
    assert("hello".startsWith("H"))
  }
}
```
Run 'test' command again:
```sh
sbt:Hello> test
```
The output is:
```sh
sbt:Hello> test
[info] Compiling 1 Scala source to /home/justribentrop_cloud/foo-build/target/scala-2.12/test-classes ...
[info] HelloSpec:
[info] - Hello should start with H *** FAILED ***
[info]   assert("hello".startsWith("H"))
[info]          |       |          |
[info]          "hello" false      "H" (HelloSpec.scala:5)
[info] Run completed in 854 milliseconds.
[info] Total number of tests run: 1
[info] Suites: completed 1, aborted 0
[info] Tests: succeeded 0, failed 1, canceled 0, ignored 0, pending 0
[info] *** 1 TEST FAILED ***
[error] Failed tests:
[error]         HelloSpec
[error] (Test / test) sbt.TestsFailedException: Tests unsuccessful
[error] Total time: 3 s, completed Jan 9, 2020 11:20:55 AM
```
#### Make the test pass 
Using an editor, change src/test/scala/HelloSpec.scala to:
```sh
import org.scalatest._

class HelloSpec extends FunSuite with DiagrammedAssertions {
  test("Hello should start with H") {
    // Hello, as opposed to hello
    assert("Hello".startsWith("H"))
  }
}
```
Confirm that the test passes, then press Enter to exit the continuous test.
#### Add sbt-native-packager plugin 
Плагин для упаковки проекта в набор: бинарник, библиотеки, start-up скрипты.
Using an editor, create project/plugins.sbt:
```sh
addSbtPlugin("com.typesafe.sbt" % "sbt-native-packager" % "1.3.4")
```
Next change build.sbt as follows to add JavaAppPackaging:
```sh
ThisBuild / scalaVersion := "2.12.10"

ThisBuild / organization := "com.example"

lazy val hello = (project in file("."))
  .enablePlugins(JavaAppPackaging)
  .settings(
    name := "Hello",
    libraryDependencies += "org.scalatest" %% "scalatest" % "3.0.5" % Test,
  )
```
#### Reload and create a .zip distribution 
Выполняем reload, чтобы загрузить JavaAppPackaging, который мы прописали на предыдущем шаге в build.sbt. 

```sh
sbt:Hello> reload
...
Создаем переносимый дистрибутив нашего приложения
```sh
sbt:Hello> dist
[info] Wrote /home/justribentrop_cloud/foo-build/target/scala-2.12/hello_2.12-0.1.0-SNAPSHOT.pom
[info] Main Scala API documentation to /home/justribentrop_cloud/foo-build/target/scala-2.12/api...
model contains 3 documentable templates
[info] Main Scala API documentation successful.
[info] Your package is ready in /home/justribentrop_cloud/foo-build/target/universal/hello-0.1.0-SNAPSHOT.zip
[success] Total time: 5 s, completed Jan 9, 2020 12:04:48 PM
```
#### Запуситить приложение из собранного дистрибутива
```sh
$ unzip -o -d /tmp/someother /tmp/foo-build/target/universal/hello-0.1.0-SNAPSHOT.zip
$ ./hello-0.1.0-SNAPSHOT/bin/hello
```
Интересно так же заголянуть в распакованный архив, посмотреть его внутренности.
## Play with sbt+Scala+Spark
#### Установить Standalone spark
#### Написать приложение на spark
#### Выполнить приложение на spark

## Play with Idea
#### Установка Idea
#### Устанвока Scala плагина
#### Создание нового scala проекта в Idea
file new project
Выбрать версии JDK, sbt, Scala с учетом последующего переноса в sbt 
Версии можно посмотреть так:
```sh
[justribentrop_cloud@sbt-scala-spark ~]$ java -version
openjdk version "1.8.0_232"
OpenJDK Runtime Environment (build 1.8.0_232-b09)
OpenJDK 64-Bit Server VM (build 25.232-b09, mixed mode)
[justribentrop_cloud@sbt-scala-spark ~]$ sbt
...
sbt:justribentrop_cloud> scalaVersion
[info] 2.12.10
sbt:justribentrop_cloud> sbtVersion
[info] 1.3.6
sbt:justribentrop_cloud> 
```
Как я понимаю, при создании нового проекта создается сущность, аналогичная запуску команды sbt в папке проекта в командной строке (см. Play with sbt). 

#### Импорт проекта из sbt в Idea
архивируем папку с проектом, вытаскиваем наружу, извлекаем из архива
натравливаем Idea на корень проекта
далее по https://www.lagomframework.com/documentation/1.5.x/java/IntellijSbtJava.html

#### Запуск, модификация и экспорт обратно в sbt

## Play with Scala+Jupiter
## Play with Scala+Spark+Jupiter

??? --- Общие вопросы ---
Не понимаю, каким образом в состав SBT включена Scala

??? --- Термины ---
Classpath
self-contained app
