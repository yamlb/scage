Introduction
------------

Scage is a framework to write simple 2D opengl games. It is written in [Scala](http://scala-lang.org/) and based on several java libraries:

 - [phys2d](http://phys2d.cokeandcode.com/) as a physics engine
 - [lwjgl](http://lwjgl.org) as an opengl wrapper
 - [slick](http://slick.cokeandcode.com/) as a resource and texture loader

The main purpose of this project is to give a convenient tool for game-developers to write a code of pure functionality without any boilerplate.

Features
--------

 - Architechture similar to actors framework with different kinds of tasks executing on different stages of app lifecycle. Simililar to actors these tasks are anonymous functions, and you can add and remove them in runtime in any scope of your app. Its all singlethreaded, so you dont have to mess with messages.
 - Vast drawing library for any kinds of 2D opengl primitives.
 - Loading and utilizing fonts from ttf-files (based on 'Slick2D' api but with improvements).
 - i18n: loading strings and even the whole interfaces from xml files. Runtime language change.
 - Framework to build in-game interfaces from xml files of simple structure.
 - App settings can be specified in a text files as a key-value pairs. Lots of engine options are set that way (alongside with the standard possibility to set them as parameters) allowing fine-tuning without app rebuilding.
 - Tracers framework: easy game objects tracking and interacting on a two-dimensional game map.
 - Lightweight wrapper upon phys2d engine.
 - Easy app building/deploing (as a standalone or via webstart) using maven infrastructure.
 - Multiple platforms support: Windows, Linux, Mac, Solaris (thanks to Java and lwjgl actually). Similar build process for any platform (with maven).
 - Client/server network api upon actors with simple text protocol based on json format.

Hello World Example
-------------------

###Rotating 'Hello World!' label

    import net.scage.ScageScreenApp
    import net.scage.ScageLib._
    import net.scage.support.Vec

    object HelloWorldExample extends ScageScreenApp("Hello World") {
      private var ang = 0f
      actionStaticPeriod(100) {
        ang += 5
      }

      backgroundColor = BLACK
      render {
        openglMove(windowSize/2)
        openglRotate(ang)
        print("Hello World!", Vec(-50, -5), GREEN)
      }
    }

!['rotating "Hello World!" demo'](http://scage.googlecode.com/svn/trunk/ScageMaven/rotating_hello.png)

###Network api example

Client sends to server random 2d vectors and server sends back corresponded normalized values.

    import net.scage.ScageApp
    import net.scage.support.net.{NetClient, NetServer}
    import net.scage.support.{Vec, State}

    object EchoExample extends ScageApp("Echo") {
      NetServer.startServer(
        port = 9800,
        onNewConnection = {
          client => client.send(State("hello" -> "send me vec and I send you back its n!"))
          (true, "")
        },
        onClientDataReceived = {
          (client, received_data) => received_data.neededKeys {
            case ("vec", vec:Vec) => client.send(State(("n" -> vec.n)))
          }
        }
      )

      NetClient.startClient(
        server_url = "localhost",
        port = 9800,
        onServerDataReceived = {
          received_data => received_data.neededKeys {
            case ("hello", hello_msg) =>
              val random_vec = Vec((math.random*100).toInt, (math.random*100).toInt)
              println("sending vec: "+random_vec)
              NetClient.send(State(("vec" -> random_vec)))
            case ("n", n:Vec) =>
              println("received n: "+n)
              println("waiting 5 sec...")
              Thread.sleep(5000)
              val random_vec = Vec((math.random*100).toInt, (math.random*100).toInt)
              println("sending vec: "+random_vec)
              NetClient.send(State("vec" -> random_vec))
          }
        }
      )

      dispose {
        NetServer.stopServer()
        NetClient.stopClient()
      }
    }
    
More examples
-------------

###Tetris

Simple tetris clone. Controls: arrow keys, spacetab - pause.

[Source Code](http://code.google.com/p/scage/source/browse/trunk/tetris/src/main/scala/su/msk/dunno/tutorials/tetris/Tetris.scala) 

![Tetris Screenshot](http://dl.dropbox.com/u/11297078/public_pics/tetris.png)

[Launch Demo](http://fzeulf.netris.ru/bor/tetris/run.jnlp)

###Snake

The Snake clone. Controls: arrow keys.

[Source Code](http://code.google.com/p/scage/source/browse/trunk/snake/src/main/scala/su/msk/dunno/scage/tutorials/snake/Snake.scala)

![Snake Screenshot](http://dl.dropbox.com/u/11297078/public_pics/snake.png)

[Launch Demo](http://fzeulf.netris.ru/bor/snake/run.jnlp)

###Arcanoid

The physics demo. Bounce ball to blocks to make them disappear and bring you points. Controls: left and right arrow keys.

[Source Code](http://code.google.com/p/scage/source/browse/#svn%2Ftrunk%2FScarcanoid%2Fsrc%2Fmain%2Fscala%2Fsu%2Fmsk%2Fdunno%2Fscar)

![Arcanoid Screenshot](http://dl.dropbox.com/u/11297078/public_pics/scaranoid.png)

[Launch Demo](http://fzeulf.netris.ru/bor/scar/run.jnlp)

###Life

The "Tracers"-framework example. Conway's Game of Life, the model of "glider gun". Press spacetab to start the process.

[Source Code](http://code.google.com/p/scage/source/browse/trunk/life/src/main/scala/su/msk/dunno/scage/tutorials/life/Life.scala)

![Life Screenshot](http://dl.dropbox.com/u/11297078/public_pics/life.png)

[Launch Demo](http://fzeulf.netris.ru/bor/life/run.jnlp)

###Uke

The physics and texture handling demo. Run and jump avoiding abysses and obstacles. Controls: Z to jump, X to destroy obstacle, down arrow key to fast land after high jump.

[Source Code](http://code.google.com/p/scage/source/browse/trunk/uke/src/main/scala/su/msk/dunno/scage/uke/Uke.scala)

![Uke Screenshot](http://dl.dropbox.com/u/11297078/public_pics/uke.png)

[Launch Demo](http://fzeulf.netris.ru/bor/uke/run.jnlp)

###Runnegun

Some kind of a top-down shooter. Shoot down circles and avoid enemy bullets. Controls: WASD to move, left mouse button to shoot.

[Source Code](http://code.google.com/p/scage/source/browse/#svn%2Ftrunk%2Frunnegun%2Fsrc%2Fmain%2Fscala%2Fsu%2Fmsk%2Fdunno%2Frunnegun)

![Runnegun Screenshot](http://dl.dropbox.com/u/11297078/public_pics/runnegun.png)

[Launch Demo](http://fzeulf.netris.ru/bor/runnegun/run.jnlp)

###Jet Flight

The texture handling example. Control a plane against very simple AI bot. Fly with left and right arrows, speed-up with up-arrow, shoot with left control.

[Source Code](https://github.com/dunnololda/scage-projects/blob/master/jetflight/src/main/scala/net/scage/tutorial/jetflight/JetFlight.scala)

![JetFlight Screenshot](https://github.com/dunnololda/scage-projects/raw/master/jetflight/jetflight.png)

[Launch Demo](http://fzeulf.netris.ru/bor/jetflight/run.jnlp)

###Blases

Shoot bubbles to reach the finish point. The game demonstrates i18n, resolution changing, one way to create game interfaces and some other things.

[Source Code](https://github.com/dunnololda/scage-projects/tree/master/blases)

![Blases Screenshot](https://github.com/dunnololda/scage-projects/raw/master/blases/blases.png)

[Launch Demo](http://fzeulf.netris.ru/bor/blases/run.jnlp)

###Pong

A very simple clone of Classic Pong (of its realization in [Nicol Examples](https://github.com/philcali/Nicol-examples/tree/master/pong) actually). Control paddles with mouse. The ball speed is slightly increased as you bounce it from the paddles.

[Source Code](https://github.com/dunnololda/scage-projects/blob/master/pong/src/main/scala/net/scage/projects/pong/Pong.scala)

![Pong Screenshot](https://github.com/dunnololda/scage-projects/raw/master/pong/pong.png)

[Launch Demo](http://fzeulf.netris.ru/bor/pong/run.jnlp)

Installation
------------

###For Maven users

Add to your pom.xml the following:

      <repositories>
      ...
          <repository>
            <id>scage</id>
            <name>Scage Maven Repo</name>
            <url>http://scage.googlecode.com/svn/maven-repository</url>
          </repository>
      </repositories>
      ...
      <dependencies>
      ...
          <dependency>
              <groupId>su.msk.dunno</groupId>
              <artifactId>scage</artifactId>
              <version>0.8</version>
              <scope>compile</scope>
          </dependency>
      </dependencies>

You can use archetype to create new scage project stub:

    $ mvn archetype:generate -DgroupId=my.company -DartifactId=app -Dversion=0.1 -Dpackage=my.company.app -DarchetypeGroupId=scage -DarchetypeArtifactId=project-archetype -DarchetypeVersion=0.8 -DarchetypeRepository=http://scage.googlecode.com/svn/maven-repository
    
To launch app from the project stub you can type:

    $ mvn clean test
    
This project stub has two profiles in its pom.xml for app building. To build a standalone app type in your console:

    $ mvn clean package -Pbuild
    
Or just:

    $ mvn clean package

as "build" is a default profile.

To build a webstart app type:

    $ mvn clean package -Pwebstart 
   
This command will create "jnlp" folder in "target". Then you can upload this folder to your host.

You also can use some IDE with good Maven and Scala support (for example, [IntelliJ IDEA](http://www.jetbrains.com/idea/)).

###For non-Maven users.

Please install Maven =) I also try to create an sbt build scenario but that would take a lot of time as my knowledge of SBT is very poor.

Feedback
--------

Feel free to ask any questions by email or using issue tracker.

