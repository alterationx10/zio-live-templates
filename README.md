This repo has some Live Templates I use in IntelliJ Idea for ZIO stuff.

## Installing the templates

It seems to be that the best way to share live templates is to
[copy a file over](https://www.jetbrains.com/help/idea/sharing-live-templates.html)

Put the file in the `templates` folder from the config folder of Idea. See
[this link](https://www.jetbrains.com/help/idea/directories-used-by-the-ide-to-store-settings-caches-plugins-and-logs.html#config-directory)
for where your config folder is:

## Live templates provided

- zio-for-comp
- zio-json-codec
- zio-json-codec-enum
- zio-json-quill-mapped-encoding

## Base example code

This is the example code we'll be showing in it's raw state, and then later, we
will show what is updated after use of the live template.

Note: The imports shown are for all the templates.

```scala
import zio._
import zio.json._
import zio.json.ast.Json
import io.getquill.MappedEncoding

object Example {

  val aZio = ???

  case class MyModel()

  object MyModel {}

  object MyEnum extends Enumeration {
    type MyEnum
    val THIS, THAT, OTHER = Value
  }

}
```

## zio-for-comp

See: `val aZio`.

This just stubs out a ZIO for-comprehension. It's my ZIO equivalent of
`val aThing = ???`

```scala
import zio._
import zio.json._
import zio.json.ast.Json
import io.getquill.MappedEncoding

object Example {

  val aZio = for {
    _ <- ZIO.attempt(???)
  } yield ???

  case class MyModel()

  object MyModel {}

  object MyEnum extends Enumeration {
    type MyEnum
    val THIS, THAT, OTHER = Value
  }

}
```

## zio-json-codec

See: `object MyModel`.

This generates a `JsonCodec` for the companion object it's run in.

```scala
import zio._
import zio.json._
import zio.json.ast.Json
import io.getquill.MappedEncoding

object Example {

  val aZio = ???

  case class MyModel()

  object MyModel {
    implicit val codec: JsonCodec[MyModel] = DeriveJsonCodec.gen[MyModel]
  }

  object MyEnum extends Enumeration {
    type MyEnum
    val THIS, THAT, OTHER = Value
  }

}
```

## zio-json-codec-enum

See: `object MyEnum`.

This generates a `JsonEncoder`, `JsonDecoder`, and `JsonCodec`for the enum
object it's run in, mapping them to to/from Strings.

```scala
import zio._
import zio.json._
import zio.json.ast.Json
import io.getquill.MappedEncoding

object Example {

  val aZio = ???

  case class MyModel()

  object MyModel {}

  object MyEnum extends Enumeration {
    type MyEnum
    val THIS, THAT, OTHER = Value
    val zEncoder: JsonEncoder[MyEnum] = JsonEncoder[String].contramap(_.toString)
    val zDecoder: JsonDecoder[MyEnum] = JsonDecoder[String].map(MyEnum.withName)
    implicit val codec: JsonCodec[MyEnum] = JsonCodec[MyEnum](zEncoder, zDecoder)
  }

}
```

## zio-json-quill-mapped-encoding

See: `object MyModel`.

This generates a `MappedEncoding` to/from the companion object it's run in to
`JSON`. Useful if you use Quill, and dump things to/from JSON columns.

Note, you would need to have run `zio-json-codec` (or already have the needed
implicit de/encoders in scope) for the `MappedEncodings` to work.

```scala
import zio._
import zio.json._
import zio.json.ast.Json
import io.getquill.MappedEncoding

object Example {

  val aZio = ???

  case class MyModel()

  object MyModel {
    implicit val codec: JsonCodec[MyModel] = DeriveJsonCodec.gen[MyModel]
    implicit val mappedEncoding: MappedEncoding[MyModel, Json] = MappedEncoding[MyModel, Json](r => r.toJsonAST match {
      case Left(msg) => throw new Exception(msg)
      case Right(json) => json
    })
    implicit val mappedDecoding: MappedEncoding[Json, MyModel] = MappedEncoding[Json, MyModel](r => r.as[MyModel] match {
      case Left(msg) => throw new Exception(msg)
      case Right(json) => json
    })
  }

  object MyEnum extends Enumeration {
    type MyEnum
    val THIS, THAT, OTHER = Value
  }

}
```
