# Recursos Funcionais da Linguagem: Iteradores e Closures

O design de Rust se inspirou em muitas linguagens e técnicas existentes, e uma 
influência significativa é a *programação funcional*. A programação em um estilo 
funcional geralmente inclui o uso de funções como valores, passando-os em argumentos, 
retornando-os de outras funções, atribuindo-os a variáveis para execução posterior, 
e assim por diante.

Neste capítulo, não discutiremos a questão do que é ou não a programação funcional, 
mas discutiremos alguns recursos de Rust que são semelhantes aos recursos em muitas 
linguagens frequentemente denominadas funcionais.

Mais especificamente, abordaremos:

* *Closures*, uma construção semelhante a uma função que você pode armazenar em uma variável
* *Iteradores*, uma maneira de processar uma série de elementos
* Como usar esses dois recursos para melhorar o projeto de E/S (I/O) no Capítulo 12
* O desempenho desses dois recursos (alerta de spoiler: eles são mais rápidos do que você imagina!)  

Outros recursos de Rust, como pattern matching (correspondência de padrões) e enums (enumerações), 
que abordamos em outros capítulos, também são influenciados pelo estilo funcional. O domínio de 
closures e iteradores é uma parte importante da escrita de código rápido Rust e idiomático; 
portanto, dedicaremos todo esse capítulo a eles.