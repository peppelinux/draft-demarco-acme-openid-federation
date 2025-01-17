# Generating the ASCII

Using `httpie`, you can generate the ASCII as follows:

```shell
http https://kroki.io/ diagram_type='plantuml' output_format='txt' diagram_source=@./plantuml
```