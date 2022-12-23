# build-mkdocs-composite-action
## prerequisites
expects Makefile in mkdocs project with the following directives:
```
build-mkdocs: env region
archive-mkdocs: env region version
download-mkdocs-assets: env region download-directory
```