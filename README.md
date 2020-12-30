# Elmを使いWebブラウザで図形をドラッグ＆ドロップで動かせるようにする
### EP18124 渡邉瑞貴
## Elm(エルム)とは
Elm は JavaScript にコンパイルできる関数型プログラミング言語でHaskellに似ている。ウェブサイトやウェブアプリケーションを作るツールという面では React のようなプロジェクトだと言える。 Elmは初心者に扱いやすく、バージョンアップが進むごとに無駄な機能が削がれるという特徴を持つ。
 * ##### 詳しいことや使い方は下のURLから参照してね
https://guide.elm-lang.jp/architecture/text_fields.html
 
## 進行状況
**ドラッグ＆ドロップさせる**
<br>
<br>
## まず始めにElmのインストールを行う。
#### 手順
#### 1　https://guide.elm-lang.jp/install.html　ここからElmをインストールする

#### 2 次にElmのコードを裏でJavaScriptに変換しながら動いているためNode.jsを事前にインストールしておく必要がある[ここから](https://nodejs.org/en/)インストールする。

#### 3 ターミナルを開き、Elmをダウンロードしたディレクトリで elm replと入力する
<br>

## 作業1週目
今回は初回ということで**簡単に扱える**[Try Elm!](https://elm-lang.org/try)で作業を行った。
https://elm-lang.org/examples ←ここにあるサンプルを参考にしながら自分で図を描いてみた。

作ったソースコード
```ruby
import Playground exposing (..)


main =
  picture
    [ rectangle brown 10 650
        |> moveDown 80
    , circle green 80
        |> moveDown 180
    , circle blue 80
        |> moveDown 20
    , circle red 80
        |> moveUp 140
    ]
```
これを実行してみると
![](https://i.imgur.com/EeyePXp.png)
こんな感じに奇妙な団子を作れた。 
なんとなく書き方が創成C,DでやったC++に似ている。

<br>

## 作業2週目(2020/11/12～11/26)
[Try Elm!](https://elm-lang.org/try)のサンプルの中にある**mouse**と**position**を組み合わせたら図形をドラッグ＆ドロップの大体の形ができると考えたのでやってみる。
```ruby
import Playground exposing (..)

main =
  game view update ()

view computer memory =
  [ circle lightPurple 30 --円の半径
      |> moveX computer.mouse.x -- マウスのx座標？
      |> moveY computer.mouse.y -- マウスのy座標?
      
  ]


update computer memory =
  if 
    computer.mouse.down
  then
    memory
  else
```
**実行結果：コンパイルは通るがクリックしても図形が表示されず思ったように起動をしない。**
![](https://i.imgur.com/ogUYjaw.png)


さらに問題点があり、このままだと図形をドラッグ＆ドロップするのではなくクリックしたところに図形がついてくる形になることである。
<br>
<br>
できている人は改善方法を教えて頂きたいです。よろしくお願いします。
## 3週目
エルムを解説してくれる動画が送られてきたのでその人の動画を参考に目標のものを作ってみる。
https://www.youtube.com/playlist?list=PLp_EUEO9JJP2P4fW-73jR3iC14476twsW
```ruby
import Browser
-- import Html exposing (Html,button,div,text)
-- 
import Svg exposing(..)
import Svg.Attributes exposing(..)
import Html.Events.Extra.Mouse as Mouse
import List.Extra

main=
  Browser.sandbox{init = init,update =update,view=view}
  
--model--

type alias RectParam={
  x: String,
  y: String
  }

type alias Model={
  rects:List(Float,Float),
  dragState: DragState}

type DragState = NotDtagging
               |Dragging Int

init:Model
init={
  rects = [(10,10),(200,200)],dragState = notDragging}

--update--
type Msg = Down Mouse.Event|Move Muse.Event
         | Up Mouse.Event

update : Msg -> Model -> Model
update msg model =
  case msg of
    Down event ->
      case intersects event.clientPos model.rects of
        Just index ->
          {model | dragState = Dragging index}
        Nothing ->
          {model | dragstate = NotDragging}
    Move event ->
      model
    
    Up event ->
     model
     
intersects: (Float,Float) -> List(Float,Float) -> Maybe Int
intersects pos rects =
  rects
   |>List.Extra.findIndex (intersect event.clientPos)

intersect:(Float,Float) -> (Float,Float) -> Bool
intersect (px,py) (rx,ry)=
  if between px rx (rx+100)then
    if between py ry (ry+100)then
      True
    else
      False
    else
     False

between: Float -> Float -> Float -> Bool
between o min max =
  if o> min then
    if o< max then
        True
    else
        False
    else
        False
        
--viewwwww--

view:Model -> Svg Msg
view model =
  svg
  [ width "500"
  , hight "500"
  , viewBox "0 0 500 500"
  , Mouse.onDown Down
  , Mouse.onMove Move
  , Mouse.onUp Up
  ]
  (List.append
  (List.map viewRect [{x="10",y="10"},{x="200",y="200"}])
  [text (Debug.toString model)]
  )
  viewRect : RectParam -> Svg msg
  viewRect param = 
          rect[x param.x , y param.y , width"100",height "100"][]
          
```
### 結果：Try Elmに導入されていないモジュールを使用しているため、できない。
![](https://i.imgur.com/rjPv4cf.png)

####  他の手段を探してみる
 - **Elmのマウスイベントについて調べよう**
https://blog.emattsan.org/entry/2019/05/26/093114
人により書き方の個性（？）が出ていて中々Elmを習得できない；；
困ったものだ

## 4週目
```ruby
import Browser
import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (..)
import Json.Decode exposing (..)

main = Browser.sandbox{init=init, view=view, update=update}

type alias Model = {x:Int, y:Int,drag:Bool}

init : Model
init = {x =0, y =0,drag=False}--動かす円の初期設定

type Msg = Down
         | Up
         | Move Int Int

update : Msg -> Model -> Model
update msg model =
  case msg of
    Down -> {model | drag = True}--押されたらしたら
    Up -> {model | drag = False}--離したら
    Move x y -> 
      {model | x= if(model.drag==True) then x else model.x,y= if(model.drag==True) then y else model.y}
        
view : Model -> Html Msg
view model =
  div[ style "position" "absolute" --動かす場所の大きさ（ウィンドウの長さ）
    ,style "background-color" "black"
    ,style "height" "100vh"
    ,style "width" "100vh"
    , on "mosuemove" (map2 Move(field "clientX" int)(field "clientY" int))
    , onMouseUp Up
  ]
  [ 
    div --動かす図形の描画
    [ style "position" "relative"
    , style "background" "blue"
    , style "top" <| String.fromInt model.y ++ "px"
    , style "left" <| String.fromInt model.x ++ "px"
    , style "width" "50px"
    , style "height" "50px"
    , onMouseDown Down 
    ]
    []
  ]
```
![](https://i.imgur.com/kxdcViX.png)
図形をドラッグ＆ドロップできるようになったがドラッグすると図形の角を持ってしまう。
offsetを使って改善してみる。
# 5週目
### ついに出来ました
```ruby
import Browser
import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (..)
import Json.Decode exposing (..)

main = Browser.sandbox{init=init, view=view, update=update}

type alias Model = {x:Int, y:Int,drag:Bool,offsetX:Int,offsetY:Int}

init : Model
init = {x =0, y =0,drag=False,offsetX=0,offsetY=0}--動かす円の初期設定

type Msg = Down
         | Up
         | Move Int Int
         | Offset Int Int

update : Msg -> Model -> Model
update msg model =
  case msg of
    Down -> {model | drag = True}--押されたらしたら
    Up -> {model | drag = False}--離したら
    Move x y -> 
      if model.drag==True  then {model | x=x - model.offsetX , y = y - model.offsetY} else model
    Offset x y -> if model.drag then model else {model| offsetX = x,offsetY = y}
                     
view : Model -> Html Msg
view model =
  div[ style "position" "absolute" --動かす場所の大きさ（ウィンドウの長さ）
    ,style "background-color" "black"
    ,style "height" "100vh"
    ,style "width" "100vw"
    , on "mousemove" (map2 Move(field "clientX" int)(field "clientY" int))
    , onMouseUp Up
  ]
  [ 
    div --動かす図形の描画
    [ style "position" "relative"
    , style "background" "blue"
    , style "top"  (String.fromInt model.y ++ "px")
    , style "left" (String.fromInt model.x ++ "px")
    , style "width" "50px"
    , style "height" "50px"
    , on "mousemove" <| (map2 Offset (field "offsetX" int)(field "offsetY" int))
    , onMouseDown Down 
    ]
    []
  ]
```
## 自分が詰まった所,次回の目標など
- if文は
    if 
        (条件式)
    then
        (条件を満たした場合の処理)
    else
        （条件を満たしていない場合の処理）
としっかり条件を満たさない場合もかかないとエラーを出すので注意する。

 - try Elmには入ってないモジュールがあるため、必要ならellyで作業する。（このプロジェクトはtry elmで完結できる）
- 書き方がとても簡単で便利と言ってもElmに対しての知識がなければ、ただ便利が不便になる。もっと勉強しようと思う。
