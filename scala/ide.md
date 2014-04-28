Scala IDE
=========

Scala IDE有三种选择：

* [Eclipse](http://scala-ide.org/download/sdk.html)
* [IntelliJ IDEA](http://www.jetbrains.com/idea/features/scala.html)
* Emacs

前两种都下载相应软件，然后更新Scala开发插件就可以了，配置都比较简单，这里主要将一下配置Emacs。


### 添加elpa站点

在init.el中添加

```lisp
;; ELPA
;; Emacs Lisp Package Archive
;;
(require 'package)
(setq package-archives '(("gnu" . "http://elpa.gnu.org/packages/")
                         ("marmalade" . "http://marmalade-repo.org/packages/")
                         ("melpa" . "http://melpa.milkbox.net/packages/")))
(package-initialize)

```

### 安装scala-mode2和ensime

```lisp
package-install <plugin>
```

### 配置

```lisp
;;==========================
;; SCALA
;;==========================

(unless (package-installed-p 'scala-mode2)
  (package-refresh-contents) (package-install 'scala-mode2))

;; load the ensime lisp code...
;; (add-to-list 'load-path "ENSIME_ROOT/src/main/elisp/")
(require 'ensime)

;; This step causes the ensime-mode to be started whenever
;; scala-mode is started for a buffer. You may have to customize this step
;; if you're not using the standard scala mode.
(add-hook 'scala-mode-hook 'ensime-scala-mode-hook)

;; Add the following to your ~/.sbt/plugins/plugins.sbt or YOUR_PROJECT/project/plugins.sbt:
;; addSbtPlugin("org.ensime" % "ensime-sbt-cmd" % "0.1.2")

```

### 建立项目文件

* 建立项目的目录和文件，请参考[STB](stb.md)
* 添加"org.ensime" % "ensime-sbt-cmd" % "0.1.2"到project/plugins.sbt中

### 生成ensime文件

```shell
sbt "ensime generate"

```

### 运行emacs并执行ensime
