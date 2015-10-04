#How to make the esphttpd Library work on Mac OSX

If you are using YUI compression you have first to install it with "brew install yuicompressor"

change the esphttpdconfig.mk in esphttpd in the yui part to:
COMPRESS_W_YUI ?= yes
YUI-COMPRESSOR ?= /usr/local/Cellar/yuicompressor/2.4.8/bin/yuicompressor

change the webpages.espfs part in the Makefile in the libesphttpd Folder to:
```
webpages.espfs: $(HTMLDIR) espfs/mkespfsimage/mkespfsimage
ifeq ("$(COMPRESS_W_YUI)","yes")
	$(Q) rm -rf html_compressed;
	$(Q) cp -r ../html html_compressed;
	$(Q) echo "Compression assets with yui-compressor. This may take a while..."
	$(Q) for file in `find html_compressed -type f -name "*.js"`; do $(YUI-COMPRESSOR) --type js $$file -o $$file; done
	$(Q) for file in `find html_compressed -type f -name "*.css"`; do $(YUI-COMPRESSOR) --type css $$file -o $$file; done
	$(Q) awk "BEGIN {printf \"YUI compression ratio was: %.2f%%\\n\", (`du -s html_compressed/ | sed 's/\([0-9]*\).*/\1/'`/`du -s ../html/ | sed 's/\([0-9]*\).*/\1/'`)*100}"
# mkespfsimage will compress html, css and js files with gzip by default if enabled
# override with -g cmdline parameter
	$(Q) cd html_compressed; find . | $(PWD)/espfs/mkespfsimage/mkespfsimage > $(PWD)/webpages.espfs; cd ..;
else
	$(Q) cd ../html; find . | $(PWD)/espfs/mkespfsimage/mkespfsimage > $(PWD)/webpages.espfs; cd ..
endif
```

##issues
the du command does not work with -b -s - had to remove this
the find command does not work with no info - had to change it to find . | instead of find |
the $(THISDIR) command does not work used $(PWD)