# Make入門
この資料では、プログラミング言語LuaのMakefileを例に、  
Makeコマンドの使い方、Makefileの書き方、読み方について説明します。  
この資料はMakeの全ての機能について説明するものではありません。  
最低限必要な情報(私が必要と思った情報)を記載します。  
もっと詳しく知りたい場合はネット、書籍などをご確認ください。

## LuaのMakefileについて

ここではVer5.3.5のソースを例に説明します。
ソースは以下からダウンロードできます。

https://www.lua.org/ftp/lua-5.3.5.tar.gz

展開すると以下のようなフォルダ構成になっています。
```
/
| - doc
| - src
     |- *.c
     |- *.h
     |- Makefile
| - Makefile
```
ルートフォルダ、srcフォルダにそれぞれMakefileがあります。  
２つのMakefileを合わせても300行程度(コメント行含む)なので、学習には最適かと思います。

## ルートフォルダのMakefileの大まかな構成

ルートフォルダのMakefileの中身は簡単に書くと以下のような構成になっています。  
実際のMakefileからコメントを削除したものを記載します。  
単純に書くと、"変数、ルール"の2つを書きます。

```Makefile
## 変数宣言部
PLAT= none

INSTALL_TOP= /usr/local
INSTALL_BIN= $(INSTALL_TOP)/bin
INSTALL_INC= $(INSTALL_TOP)/include
INSTALL_LIB= $(INSTALL_TOP)/lib
INSTALL_MAN= $(INSTALL_TOP)/man/man1
INSTALL_LMOD= $(INSTALL_TOP)/share/lua/$V
INSTALL_CMOD= $(INSTALL_TOP)/lib/lua/$V

INSTALL= install -p
INSTALL_EXEC= $(INSTALL) -m 0755
INSTALL_DATA= $(INSTALL) -m 0644
MKDIR= mkdir -p
RM= rm -f

PLATS= aix bsd c89 freebsd generic linux macosx mingw posix solaris
TO_BIN= lua luac
TO_INC= lua.h luaconf.h lualib.h lauxlib.h lua.hpp
TO_LIB= liblua.a
TO_MAN= lua.1 luac.1

V= 5.3
R= $V.4

## ルール
all:	$(PLAT)

$(PLATS) clean:
	cd src && $(MAKE) $@

test:	dummy
	src/lua -v

install: dummy
	cd src && $(MKDIR) $(INSTALL_BIN) $(INSTALL_INC) $(INSTALL_LIB) $(INSTALL_MAN) $(INSTALL_LMOD) $(INSTALL_CMOD)
	cd src && $(INSTALL_EXEC) $(TO_BIN) $(INSTALL_BIN)
	cd src && $(INSTALL_DATA) $(TO_INC) $(INSTALL_INC)
	cd src && $(INSTALL_DATA) $(TO_LIB) $(INSTALL_LIB)
	cd doc && $(INSTALL_DATA) $(TO_MAN) $(INSTALL_MAN)

uninstall:
	cd src && cd $(INSTALL_BIN) && $(RM) $(TO_BIN)
	cd src && cd $(INSTALL_INC) && $(RM) $(TO_INC)
	cd src && cd $(INSTALL_LIB) && $(RM) $(TO_LIB)
	cd doc && cd $(INSTALL_MAN) && $(RM) $(TO_MAN)

local:
	$(MAKE) install INSTALL_TOP=../install

none:
	@echo "Please do 'make PLATFORM' where PLATFORM is one of these:"
	@echo "   $(PLATS)"
	@echo "See doc/readme.html for complete instructions."

dummy:

echo:
	@cd src && $(MAKE) -s echo
	@echo "PLAT= $(PLAT)"
	@echo "V= $V"
	@echo "R= $R"
	@echo "TO_BIN= $(TO_BIN)"
	@echo "TO_INC= $(TO_INC)"
	@echo "TO_LIB= $(TO_LIB)"
	@echo "TO_MAN= $(TO_MAN)"
	@echo "INSTALL_TOP= $(INSTALL_TOP)"
	@echo "INSTALL_BIN= $(INSTALL_BIN)"
	@echo "INSTALL_INC= $(INSTALL_INC)"
	@echo "INSTALL_LIB= $(INSTALL_LIB)"
	@echo "INSTALL_MAN= $(INSTALL_MAN)"
	@echo "INSTALL_LMOD= $(INSTALL_LMOD)"
	@echo "INSTALL_CMOD= $(INSTALL_CMOD)"
	@echo "INSTALL_EXEC= $(INSTALL_EXEC)"
	@echo "INSTALL_DATA= $(INSTALL_DATA)"

pc:
	@echo "version=$R"
	@echo "prefix=$(INSTALL_TOP)"
	@echo "libdir=$(INSTALL_LIB)"
	@echo "includedir=$(INSTALL_INC)"

.PHONY: all $(PLATS) clean test install local none dummy echo pecho lecho
```

### 変数

変数については以下の記載が参考になります。

```Makefile
INSTALL_TOP= /usr/local         # <--- INSTALL_TOP /usr/local が入る
INSTALL_BIN= $(INSTALL_TOP)/bin # <--- INSTALL_BINには /usr/local/bin が入る
```

変数の展開は上記のように"$(変数名)"と先頭に$をつけ、括弧で囲みます。  
ただし、変数名が１文字の時は少し書き方が異なります。  
下記のように変数が１文字の場合(ここでは変数名 V, R)、展開するときは変数名を括弧で囲む必要はありません。

```Makefile
# 変数V, Rに値を設定
V= 5.3
R= $V.4
# ...
echo:
	@cd src && $(MAKE) -s echo
	@echo "PLAT= $(PLAT)"
	@echo "V= $V" # <---+-- 変数名を括弧で囲む必要はない
	@echo "R= $R" # <---|
# ...
```

### ルール
LuaのMakefileを見る前に、Makefileのルールの書き方をいかに記載します。
```
[ターゲット] [:(コロン)] [必須項目(複数可)]
[TAB] [コマンド]
```

それぞれの項目の意味は以下になります。

| 名称        | 意味           |
| ------------- |-------------  |
| [ターゲット(複数可)] [:(コロン)]     | 作成したいファイル名。スペース区切りで複数並べることができます。最後に:(コロン)が必要です。 |
| [必須項目(複数可)]     | ターゲットの作成に必要なファイル。ない場合は空欄。スペース区切りで複数並べることができます。 |
| [TAB] [コマンド]     | ターゲットを作成するコマンド。先頭にTABが必要になります。 |


Makefileの中に以下のような記載があります。

```Makefile
PLATS= aix bsd c89 freebsd generic linux macosx mingw posix solaris
$(PLATS) clean:
	cd src && $(MAKE) $@ # <--- 変数 MAKE, @ については後述
```

変数を使用していて少しわかりにくいのですが、これは変数を展開した形で書くと以下のようになります。

```Makefile
aix bsd c89 freebsd generic linux macosx mingw posix solaris clean:
	cd src && $(MAKE) $@
```

さらに省略しないで書くと以下になります。  
注意して見て欲しいのですが、変数@を展開した形で記載しています。  
変数@はターゲット名が入ります

```Makefile
aix:
	cd src && $(MAKE) aix # <--- $@ を展開した結果
bsd:
	cd src && $(MAKE) bsd # <--- $@ を展開した結果
c89:
	cd src && $(MAKE) c89 # <--- $@ を展開した結果
# ...
clean:
	cd src && $(MAKE) clean # <--- $@ を展開した結果
```

例えばルートフォルダで
```
make aix
```
と入力すると、ターゲット名がaixのコマンドが実行されます。
```Makefile
aix:
	cd src && $(MAKE) aix # <--- make aixの場合、このコマンドが実行される
```
この場合、 srcフォルダに移動した後に make aixコマンドが実行されます。  
変数 MAKE については後述します。いまは makeコマンドと同等の意味と捉えていて構いません。


## srcフォルダのMakefileの大まかな構成

srcフォルダの中のMakefileについても基本的にはルートフォルダのものと作りは同じです。
以下にルートフォルダから "make aix"と呼ばれた場合を順を追って説明します。


まず、ターゲット名aixのコマンドが実行されます。
```Makefile
aix:
	$(MAKE) $(ALL) CC="xlc" CFLAGS="-O2 -DLUA_USE_POSIX -DLUA_USE_DLOPEN" SYSLIBS="-ldl" SYSLDFLAGS="-brtl -bexpall"
```
もう一度 makeコマンド"make all"が実行されます。  
"make all"のあとに"CC="xlc" CFLAGS="-O2 -DLUA_USE_POSIX -DLUA_USE_DLOPEN" ..." とコマンドが続いていますが、これは  
変数の更新を行なっています。ここでは CC, CFLAGS, SYSLIBS, SYSLDFLAGSの4つの変数を更新しています。  
Makefileの中にこれらの変数がありますが、make を行う時に更新されることになります。

```Makefile
CC= gcc -std=gnu99
CFLAGS= -O2 -Wall -Wextra -DLUA_COMPAT_5_2 $(SYSCFLAGS) $(MYCFLAGS)
#...
SYSLDFLAGS=
SYSLIBS=
```

次はターゲット名 all が実行されます。  
ここから変数が多く使用されていますが順に見ていきます。  
処理される順番は簡単に書くと、以下のコメントのように[1][2][3][4]となります。

```Makefile
ALL_T= $(LUA_A) $(LUA_T) $(LUAC_T)
# ...
$(LUA_A): $(BASE_O) # <--- [2]
	$(AR) $@ $(BASE_O)
	$(RANLIB) $@

$(LUA_T): $(LUA_O) $(LUA_A) # <--- [3]
	$(CC) -o $@ $(LDFLAGS) $(LUA_O) $(LUA_A) $(LIBS)

$(LUAC_T): $(LUAC_O) $(LUA_A) # <--- [4]
	$(CC) -o $@ $(LDFLAGS) $(LUAC_O) $(LUA_A) $(LIBS)
# ...
all:
	$(ALL_T) # <--- [1]
```

変数が多く見にくいので、[2]の

```Makefile
$(LUA_A): $(BASE_O) # <--- [2]
	$(AR) $@ $(BASE_O)
	$(RANLIB) $@
```
を変数を展開した形にすると以下のようになります。

```Makefile
liblua.a: lapi.o lcode.o lctype.o ldebug.o ldo.o ldump.o lfunc.o lgc.o llex.o \
	lmem.o lobject.o lopcodes.o lparser.o lstate.o lstring.o ltable.o \
	ltm.o lundump.o lvm.o lzio.o　lauxlib.o lbaselib.o lbitlib.o lcorolib.o ldblib.o liolib.o \
	lmathlib.o loslib.o lstrlib.o ltablib.o lutf8lib.o loadlib.o linit.o

	ar rcu liblua.a lapi.o lcode.o lctype.o ldebug.o ldo.o ldump.o lfunc.o lgc.o llex.o \
	lmem.o lobject.o lopcodes.o lparser.o lstate.o lstring.o ltable.o \
	ltm.o lundump.o lvm.o lzio.o　lauxlib.o lbaselib.o lbitlib.o lcorolib.o ldblib.o liolib.o \
	lmathlib.o loslib.o lstrlib.o ltablib.o lutf8lib.o loadlib.o linit.o

	ranlib liblua.a
```

これを見ると、変数を使用している理由がわかるかと思います。
変数を使用しない場合、Makefileのメンテナンスが大変です。


## 変数 MAKE について

ルートフォルダのMakefileの中でいかのような記載があり、MAKE という変数を使用しています。

```Makefile
PLATS= aix bsd c89 freebsd generic linux macosx mingw posix solaris
$(PLATS) clean:
	cd src && $(MAKE) $@
```

上記の例では src フォルダに移動し、そこで make コマンドを実行することになります。  
$(MAKE) とすることで、makeコマンドをオプション付きで実行をした時に、  
オプション付きでmakeが実行されます。例えば、ルートフォルダで

```
$ make --just-print aix 
```

と実行した場合は MAKE 変数が展開されると

```Makefile
PLATS= aix bsd c89 freebsd generic linux macosx mingw posix solaris
$(PLATS) clean:
	cd src && make --just-print $@ # <--- $(MAKE)が展開され、 "make --just-print" となる
```

というように、追加したオプションも引き継がれます。  
この説明は少し正確ではなく、実際はオプションの情報(ここでは --just-print)は MAKEFLAGS変数に  
"--just-print"の情報が保持され、makeコマンドが実行される、という流れになります。

# まとめ
 * Makefileの中身は、変数へ値の設定、ルールの記載が主となる。
 * 変数の展開は $(変数名) とする。ただし1文字の変数名については括弧は不要
 * MAKE 変数については $(MAKE) と書いた場合はオプションの情報も引き継がれる
 * 変数 @ はターゲット名が入る

## 参考文献
GNU Make  
https://www.oreilly.co.jp/books/4873112699/




