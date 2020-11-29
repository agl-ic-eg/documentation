---
title: Qt AFB Websocket client
---

## QAfbWebsocketClient(QObject* parent = nullptr)

Default constructor.

* `parent`: Parent object.

## QAbstractSocket::SocketError error()

Get and return the last error code.

## QString errorString()

Get and return the last error as a string.

## bool isValid()

Check if connection is ready or not.

Returns `true` if the connected is ready to read and write, `false` otherwise.

## void call(const QString& api, const QString& verb, const QJsonValue& arg = QJsonValue(), closure_t closure = nullptr)

Call an api's verb with an argument.

* `api`: Api to call.
* `verb`: Verb to call.
* `arg`: Argument to pass.
* `closure`: callback function to call at the verb reply

## void QAfbWebsocketClient::sendTextMessage(QString msg)

Send a text message over the websocket.

This is use for test only, you should not use this method because it sent text
**as-is**, so you have to follow the binder's protocol by your self.

* `msg`: Message to send.