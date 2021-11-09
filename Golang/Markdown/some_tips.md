# Some Tips

Golang 相关的小技巧

## 1.1 go list -m

go list -m  用于列出当前模组引用的依赖

```sh
go list -m all //列出所有
GoWebAppNotes
github.com/davecgh/go-spew v1.1.1
github.com/go-sql-driver/mysql v1.6.0
github.com/gomodule/redigo v1.8.4
github.com/pmezard/go-difflib v1.0.0
github.com/sirupsen/logrus v1.8.1
github.com/stretchr/objx v0.1.0
github.com/stretchr/testify v1.5.1
golang.org/x/sys v0.0.0-20191026070338-33540a1f6037
gopkg.in/check.v1 v0.0.0-20161208181325-20d25e280405
gopkg.in/yaml.v2 v2.2.2
```

go list -m -versoions module_name 列出指定依赖可用的版本

```sh
go list -m -versions github.com/go-sql-driver/mysql                   
github.com/go-sql-driver/mysql v1.0.0 v1.0.1 v1.0.2 v1.0.3 v1.1.0 v1.2.0 v1.3.0 v1.4.0 v1.4.1 v1.5.0 v1.6.0
```

## 1.2 why error strings should not be capitalized

> StringsError Error strings should not be capitalized (unless beginning with proper nouns or acronyms) or end with punctuation, since they are usually printed following other context. That is, use fmt.Errorf("something bad") not fmt.Errorf("Something bad"), so that log.Printf("Reading %s: %v", filename, err) formats without a spurious capital letter mid-message. This does not apply to logging, which is implicitly line-oriented and not combined inside other messages.

