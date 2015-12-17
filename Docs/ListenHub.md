Listen Hub
===========

## 概要
このHubでは、主にVSShareのViewerに必要な関数群および通知機能を提供しています。
現在、通信方式はSignalRのみをサポートしています。
本ドキュメントで用いられている単語・用語に関しては、`Basics.md`をご覧ください。

## 通信の流れ

1. エンドポイントへ接続する。
2. 接続完了後、`Authorize`メソッドをInvokeする。認証が正しく行われれば、認証成功レスポンスが返される。
3. 現在のセッションリストの取得やコンテンツの取得など、初期化処理を行う。
この時点から、各種関数群の呼び出し、および通知の受け取りが可能となる。
4. 接続を終了したい時は、Hub自体をDisconnectすることで、自動的にログアウト処理が行われる。

## 呼び出し可能関数
**注意**：Authorizeメソッド以外は認証が行われるまで呼び出すことが出来ません。


### Authorize(request)
#### 概要
認証を行います。

#### 引数
* **request**  
```json
{
	token: "1234567890abcdef" // 認証用のワンタイムトークを指定します
}
```

#### 戻り値
```json
{
	success: true // 認証結果を示します
}
```


### GetSessionList()
#### 概要
現在配信されているセッション一覧情報を取得します。

#### 引数
なし

#### 戻り値
```json
[
	{
		id: "1234567890abcdef", // セッションIDを示します
		filename: "C:¥Foo¥Program.cs", // 現在編集中のドキュメントのファイル名を示します
		type: 1, // 編集中の言語を示します。モデル：ContentType
		owner_name: "bonprosoft" // 配信者の名前を示します
	}
]
```


### GetSessionInfo(request)
#### 概要
指定したセッションに関する情報を取得します。

#### 引数
* **request**  
```json
{
	id: "1234567890abcdef" // セッションIDを指定します
}
```

#### 戻り値
```json
{
	id: "1234567890abcdef", // セッションIDを示します
	filename: "C:¥Foo¥Program.cs", // 現在編集中のドキュメントのファイル名を示します
	type: 1, // 編集中の言語を示します。モデル：ContentType
	owner_name: "bonprosof" // 配信者の名前を示します
}
```


### GetSessionContent(request)
#### 概要
指定したセッションのコンテンツを取得します。

#### 引数
* **request**  
```json
{
	id: "1234567890abcdef" // セッションIDを指定します
}
```

#### 戻り値
```json
{
	id: "1234567890abcdef", // セッションIDを示します
	data: [] // 現在のコンテンツの情報を返します。モデル：UpdateContentData[]
}
```


### GetSessionCursor(request)
#### 概要
指定したセッションのカーソル情報を取得します。

#### 引数
* **request**  
```json
{
	id: "1234567890abcdef" // セッションIDを指定します
}
```

#### 戻り値
```json
{
	active: {}, // 現在のカーソル位置を示します。モデル：CursorPosition
	anchor: {}, // 選択開始位置を示します。この値はtypeがCursorType.Selectの時のみ有効です。モデル：CursorPosition
	type: 0 // カーソルモードを示します。モデル：CursorType
}
```

## 実装が必要な関数
### UpdateRoomStatus(item)
アクティブな閲覧者数、部屋の訪問数を通知します。

#### 引数
* **item**
```json
{
	view: 3, // 現在アクティブな閲覧者数を示します
	visitor: 2000 // 現在の配信までのトータル訪問者数を示します
}
```


### AppendSession(item)
新しいセッションが追加されたことを通知します。

#### 引数
* **item**
```json
{
	id: "1234567890abcdef", // セッションIDを示します
	filename: "C:¥Foo¥Program.cs", // 現在編集中のドキュメントのファイル名を示します
	type: 1, // 編集中の言語を示します。モデル：ContentType
	owner_name: "bonprosoft" // 配信者の名前を示します
}
```


### RemoveSession(item)
セッションが削除されたことを通知します。

#### 引数
* **item**
```json
{
	id: "1234567890abcdef" // セッションIDを示します
}
```


### UpdateSessionInfo(item)
セッション情報が更新されたことを通知します。

#### 引数
* **item**
```json
{
	id: "1234567890abcdef", // セッションIDを示します
	filename: "C:¥Foo¥Program.cs", // 現在編集中のドキュメントのファイル名を示します
	type: 1, // 編集中の言語を示します。モデル：ContentType
	owner_name: "bonprosoft" // 配信者の名前を示します
}
```


### UpdateSessionContent(item)
セッションのコンテンツが更新されたことを示します。

#### 引数
* **item**
```json
{
	id: "1234567890abcdef", // セッションIDを示します
	data: [] // 現在のコンテンツの情報を返します。モデル：UpdateContentData[]
}
```


### UpdateSessionCursor(item)
セッションのカーソル位置が更新されたことを示します。

#### 引数
* **item**
```json
{
	active: {}, // 現在のカーソル位置を示します。モデル：CursorPosition
	anchor: {}, // 選択開始位置を示します。この値はtypeがCursorType.Selectの時のみ有効です。モデル：CursorPosition
	type: 0 // カーソルモードを示します。モデル：CursorType
}
```

## モデルリファレンス
### UpdateContentData
コンテンツのアップデート差分を示します。
```json
{
	data: [], // アップデート差分データを示します。モデル：Line[]
	type: 0, // アップデート差分の種類を示します。モデル：UpdateType
	pos: 10, // アップデート差分の開始行番号を示します
	len: 1, // アップデート差分の、開始行番号からの適用範囲を示します
	order: 0 /* アップデート差分の優先順位を示します。小さいほうが優先的に適用されます。
	// これはUpdateContentDataがリストになって配信される場合に、適用順序を決定する際に有効です。*/
}
```

### UpdateType
アップデート差分の種類を示します。
```csharp
public enum UpdateType
{
	Insert = 0,
	Delete = 1,
	Replace = 2,
	Append = 3,
	RemoveMarker = 4,
	ResetAll = 5
}
```

### Line
行単位の差分データを示します。
```json
{
	text: "Lorem ipsum dolor sit amet", // テキスト情報を示します
	modified: true // 編集マーカーが付与されているかを示します	
}
```

### ContentType
編集中の言語を示します。

```csharp
public enum ContentType
{
	PlainText = 0,
	CSharp = 1,
	VB_NET = 2,
	XML = 3,
	JSON = 4
}
```

### CursorType
現在のカーソルの種類を示します。
```csharp
public enum CursorType
{
	Point = 0, // ある箇所にカーソルがある状態
	Select = 1 // 範囲選択されている状態
}
```

### CursorPosition
カーソルの位置を示します。
```json
{
	line: 10, // 行番号を示します
	pos: 15 // 列番号(行内の桁番号)を示します
}
```

