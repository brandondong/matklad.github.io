VERSION=0.1.0
REVISION=1
ROCKSPEC=djot-$(VERSION)-$(REVISION).rockspec
MODULES=djot/match.lua djot/attributes.lua djot/inline.lua djot/block.lua djot/ast.lua djot/emoji.lua djot/html.lua djot/filter.lua djot.lua
SOURCES=$(MODULES) bin/main.lua
TESTSOURCES=test.lua pathological_tests.lua
LIBLUA=/usr/local/lib/libluajit.a
LUAHEADERS=/usr/local/include/luajit-2.0
LUAOPTIONS=-O2
BUNDLE=djot
VIMDIR?=~/.vim
all: test doc/syntax.html

test: $(ROCKSPEC)
	luarocks test
.PHONY: test

testall: test pathological fuzz
.PHONY: testall

fuzz:
	lua fuzz.lua
.PHONY: fuzz

pathological:
	@perl -e 'alarm shift; exec @ARGV' 10 lua pathological_tests.lua
.PHONY: pathological

bench: m.dj
	du -h m.dj
	-bench "lua bin/main.lua m.dj"
.PHONY: bench

m.dj:
	pandoc -t djot-writer.lua https://raw.githubusercontent.com/jgm/pandoc/2.18/MANUAL.txt -o m.dj

linecount:
	wc -l $(SOURCES)
.PHONY: linecount

check:
	luacheck $(SOURCES) $(TESTSOURCES)
.PHONY: check

djotbin:
	luastatic bin/main.lua $(MODULES) $(LIBLUA) -I$(LUAHEADERS) $(LUAOPTIONS) -pagezero_size 10000 -o djotbin

# Single-file version of library
djot-complete.lua: djot.lua
	lua -lamalg $<
	amalg.lua -o $@ -s $< -c

doc/syntax.html: doc/syntax.md
	pandoc --lua-filter doc/code-examples.lua $< -t html -o $@ -s --css doc/syntax.css --self-contained --wrap=preserve --toc --section-divs -Vpagetitle="Djot syntax reference"

# luarocks packaging

install: $(ROCKSPEC)
	luarocks make $(ROCKSPEC)
.PHONY: install

rock: $(ROCKSPEC)
	luarocks --local make $(ROCKSPEC)
.PHONY: rock

vim:
	cp editors/vim/syntax/djot.vim $(VIMDIR)/syntax/
	cp editors/vim/ftdetect/djot.vim $(VIMDIR)/ftdetect/
.PHONY: vim

$(ROCKSPEC): rockspec.in
	sed -e "s/_VERSION/$(VERSION)/g; s/_REVISION/$(REVISION)/g" $< > $@


