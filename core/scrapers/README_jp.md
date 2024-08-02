汎用ページパーサーを提供しており、このパーサーは信頼できるソースから記事リストをインテリジェントに取得します。各記事URLに対して、まず `gne` を使用して解析を試み、失敗した場合は `llm` を使用して解析します。

このソリューションにより、ほとんどの一般的なニュースサイトやポータルサイトからの情報をスキャンして抽出することができます。

**しかし、より理想的かつ効率的なスキャンを実現するために、ユーザー自身のビジネスシナリオに応じた特定のソース専用のパーサーを開発することを強くお勧めします。**

また、WeChat 公共アカウントの記事（mp.weixin.qq.com）に特化したパーサーも提供しています。

**特定のソース専用に開発したパーサーをこのリポジトリに貢献していただける場合は、大変感謝いたします！**

## 特定ソースパーサー開発規範

### 規範

**覚えておいてください:それは非同期関数でなければなりません**

1. **パーサーは、記事リストページと記事詳細ページをインテリジェントに区別できる必要があります。**
2. **パーサーの入力パラメーターは `url` と `logger` のみを含むべきです：**
   - `url` はソースの完全なアドレス（`str` タイプ）
   - `logger` はロギングオブジェクト（専用のロガーを構成しないでください）
3. **パーサーの出力は `flag` と `result` を含み、形式は `tuple[int, Union[set, dict]]`：**
   - `url` が記事リストページの場合、`flag` は `1` を返し、`result` はすべての記事ページURLのコレクション（`set`）を返します。
   - `url` が記事ページの場合、`flag` は `11` を返し、`result` はすべての記事詳細（`dict`）を返します。形式は以下の通りです：

     ```python
     {'url': str, 'title': str, 'author': str, 'publish_time': str, 'content': str, 'abstract': str, 'images': [str]}
     ```

     _注意：`title` と `content` は空であってはなりません。_

     **注意：`publish_time` の形式は `"%Y%m%d"`（日付のみ、`-` はなし）である必要があります。スクレイパーが取得できない場合は、当日の日付を使用してください。**

   - 解析に失敗した場合、`flag` は `0` を返し、`result` は空の辞書 `{}` を返します。

     _`pipeline` は `flag` 0 を受け取ると他の解析ソリューション（存在する場合）を試みます。_

   - ページの取得に失敗した場合（例えば、ネットワークの問題）、`flag` は `-7` を返し、`result` は空の辞書 `{}` を返します。

     _`pipeline` は `flag` -7 を受け取ると、同一プロセス内では再解析を試みません。_

### 登録

スクレイパーを作成したら、このフォルダにプログラムを配置し、`__init__.py` の `scraper_map` にスクレイパーを次のように登録してください：

```python
{'domain': 'スクレイパー関数名'}
```

domain の取得には urllib.parse を使用することをお勧めします：

```python
from urllib.parse import urlparse

parsed_url = urlparse("l'URL du site")
domain = parsed_url.netloc
```