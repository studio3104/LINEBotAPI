#LINEBotAPI

[![Swift][swift-badge]][swift-url]
[![Platform][platform-badge]][platform-url]
[![License][mit-badge]][mit-url]
[![Bot][linebot-badge]][linebot-url]

## Overview

**LINEBotAPI** is a SDK of the LINE BOT API Trial for Swift.

- Swift 3 support
- Using [Zewo](https://github.com/Zewo/Zewo)
- Linux Ready

## Features

- [x] Send text/image/video/audio/location/sticker message
- [x] Send multiple messages
- [x] Send rich message
- [x] Handle received operation(add as friend or blocked)

## A Work In progress

LINEBotAPI is currently in development.

## Attention

Currently LINEBotAPI works with `DEVELOPMENT-SNAPSHOT-2016-04-12-a`.

## Getting started

### Installation

Before we start, we need to install some tools and dependencies.

### swiftenv

[`swiftenv`](https://github.com/kylef/swiftenv) allows you to easily install multiple versions of swift.

```
% git clone https://github.com/kylef/swiftenv.git ~/.swiftenv
```
and add settings for your shell(For more information, please see [swiftenv's wiki](https://github.com/kylef/swiftenv)).

### Swift

Install swift using swiftenv.

```
% swiftenv install DEVELOPMENT-SNAPSHOT-2016-04-12-a
```

### Install Zewo

Install libraries depend on [Zewo](https://github.com/Zewo/Zewo).

- On OS X

    Install Zewo dependencies
    ```
    % brew install zewo/tap/zewo
    ```

- On Linux

    Install Zewo dependencies

    ```
    % echo "deb [trusted=yes] http://apt.zewo.io/deb ./" | sudo tee --append /etc/apt/sources.list
    % sudo apt-get update
    % sudo apt-get install zewo
    ```

    Install Clang and ICU

    ```
    % sudo apt-get install clang libicu-dev
    ```

## Install other libraries.

- On OS X

    ```
    % brew install openssl curl
    % brew link --force openssl
    ```

- On Linux

    ```
    % sudo apt-get install build-essential libcurl4-openssl-dev
    ```

# Create project

We got prepared for a creating project.

Let's create your LINE Bot!

## Make project directory.

```
% mkdir linebot && cd linebot
```

Set Swift version to `DEVELOPMENT-SNAPSHOT-2016-04-12-a`.

```
% swiftenv local DEVELOPMENT-SNAPSHOT-2016-04-12-a
```

Initialize project directory with swift package manager(**SPM**).

```
% swift build --init
```

Then this command will create the basic structure for app.

```
.
├── Package.swift
├── Sources
│   └── main.swift
└── Tests
```

## Package.swift

Open `Package.swift` and make it looks like this:

```swift
import PackageDescription

let package = Package(
    name: "linebot",
    dependencies: [
        .Package(url: "https://github.com/yoshik/LINEBotAPI.git", majorVersion: 0, minor: 1),
    ]
)
```

## main.swift

Next, write main program in `main.swift`.

```swift
import LINEBotAPI

do {
    if let mid = getVar(name: "TO_MID") {
        let bot = try LINEBotAPI()
        try bot.sendText(to: mid, text: "Hello! Hello!")
    } else {
        print("set env TO_MID")
    }
} catch let e {
    print(e)
}
```

This code:
- get target `mid`(A user id on LINE)
- send text to `mid` specified.

## Build project

Change lib and include paths to your environment.

```
% swift build -Xlinker -L/usr/local/lib -Xcc -I/usr/local/include -Xswiftc -I/usr/local/include
```

>On Linux only, currently, [venice](https://github.com/VeniceX/Venice)'s directory structure is invalid. So you must run following command only once before build. For more information, see LINEBotAPI's `Makefile`.

```
# fetch dependencies only.
% swift build --fetch

# and move source files to upper directory.
% mv ./Packages/Venice-*/Source/Venice/*/* ./Packages/Venice-*/Source/
% rm -fr ./Packages/Venice-*/Source/Venice
```

## Run it

After it compiles, run it.

>Your must specify `LINE_CHANNEL_ID`, `LINE_CHANNEL_SECRET`, `LINE_BOT_MID` and `TO_MID` to yours. `TO_MID` is your mid.

```
% env LINE_CHANNEL_ID=XXXXXXXXX LINE_CHANNEL_SECRET=XXXXXXXXXXXXXXXXXXXXXXXXXXXXX LINE_BOT_MID=XXXXXXXXXXXXXXXXXXXXXXXXXXXXX TO_MID=XXXXXXXXXXXXXXXXXXXXXXXXXXXXX .build/debug/linebot
```

You will get a message from bot on LINE if you had setup setting on bot management page.

> If you got an error `.build/debug/linebot: error while loading shared libraries: libCLibvenice.so: cannot open shared object file: No such file or directory`, add `.build/debug/` to your `LD_LIBRARY_PATH`.

```
% export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:.build/debug/
```

# Start Server

LINEBotAPI allows you to start server using `Zewo`.

## main.swift

Open `main.swift` and make it look like this:

```swift
import LINEBotAPI
import HTTPServer
import Router

do {
    let bot = try LINEBotAPI()

    // Initializer a router.
    let router = Router { (route) in
        // Waiting for POST request on /linebot/callback.
        route.post("/linebot/callback") { request -> Response in
            // Parsing request and validate signature.
            return try bot.parseRequest(request) { content in
                // Processing each messages or operations.
                // The contents can be enqueued to external storages as you wish.
                if let content = try ContentParser.parse(content) {
                    if let message = content as? TextMessage,
                        mid = message.fromMid {
                        // send text if content is a message.
                        try bot.sendText(to: mid, text: "Hi! Hi!")
                    } else if let op = content as? Operation,
                        mid = op.fromMid,
                        type = op.opType where type = .Added {
                        // send text if content is a operation to add.
                        try bot.sendText(to; mid, text: "Thanks to add as friend")
                    }
                }
            }
        }
    }

    // start server
    try Server(router).start()
} catch let e {
    print(e)
}
```

Then build and run it.

```
% env LINE_CHANNEL_ID=XXXXXXXXX LINE_CHANNEL_SECRET=XXXXXXXXXXXXXXXXXXXXXXXXXXXXX LINE_BOT_MID=XXXXXXXXXXXXXXXXXXXXXXXXXXXXX .build/debug/linebot
```

The server will be started on port 8080.

This server will be waiting a POST request from Bot Server at `/linebot/callback`.

# Other examples

## Send multiple messages

```swift
try bot.sendMultipleMessage { builder in
    builder.addText(text: "Hello, bot!")
    builder.addImage(
        imageUrl: "http://www.example.com/some.jpg",
        previewUrl: "http://www.example.com/some-thumb.jpg",
    )
}
```

## Send rich messages

```swift
try bot.sendRichMessage(to: mid, imageUrl: "http://example.com/images", altText: "Text for low-level devices") { (builder) in
    let action0 = RichMessageAction(name: "openHomepage", text: "Open Homepage", linkUri: "http://example.com/homepage")
    let listener0 = RichMessageListener(bounds: Bounds(x: 0, y: 0, width: 1040, height: 520), action: action0)
    builder.addListener(listener: listener0)

    let action1 = RichMessageAction(name: "showItem", text: "Show Item", linkUri: "http://example.com/showItem")
    let listener1 = RichMessageListener(bounds: Bounds(x: 0, y: 520, width: 520, height: 520), action: action1)
    builder.addListener(listener: listener1)

    let action2 = RichMessageAction(name: "search", text: "Search", linkUri: "http://example.com/search")
    let listener2 = RichMessageListener(bounds: Bounds(x: 520, y: 520, width: 520, height: 520), action: action2)
    builder.addListener(listener: listener2)
}
```

`imageUrl` must serve images of some size(For more information, see [official docs](https://developers.line.me/bot-api/api-reference#sending_rich_content_message_prerequisite)).

`width` must be 1040px for `Bounds()` and a maximum size of `height` must be less than 2080px.

# Tips

## Can I develop on Xcode?

Yes, sure. You can generate a xcode project file with following command.

```
% swift build -Xlinker -L/usr/local/lib -Xcc -I/usr/local/include -Xswiftc -I/usr/local/include -X
```

## Can I use https server?

Maybe. We are developing it using reverse proxy, but you must be able to support https because Zewo has `HTTPSServer`.

# License

LINEBotAPI is released under the MIT license. See LICENSE for details.

[swift-badge]: https://img.shields.io/badge/Swift-3.0-orange.svg?style=flat
[swift-url]: https://swift.org
[platform-badge]: https://img.shields.io/badge/Platform-Mac%20%26%20Linux-lightgray.svg?style=flat
[platform-url]: https://swift.org
[mit-badge]: https://img.shields.io/badge/License-MIT-blue.svg?style=flat
[mit-url]: https://tldrlegal.com/license/mit-license
[linebot-badge]:https://img.shields.io/badge/Bot-LINE-brightgreen.svg?style=flat
[linebot-url]:https://developers.line.me/bot-api/overview
