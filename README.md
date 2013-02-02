SimpleCompression
=================

A simple compression program that uses Huffman coding.

This is the report (in Portuguse) that was part of the project.

# Como utilizar
Para compilar o código, rode o seguinte comando (em sistemas Unix):
```
gcc hf.c -o hf
```
Um executável com nome `hf` é gerado.

A interface em linha de comando é a sugerida na especificação do projeto. Para 
mais detalhes, basta executar o arquivo sem nenhum argumento que a sintaxe 
esperada será mostrada.
Qualquer arquivo é aceito (não apenas arquivos de texto, mas também binários).


# Breve descrição da implementação
O código foi escrito em C.

São utilizadas duas estruturas: `Node`, para representar um nó na árvore de Huffman e 
`Code`, que guarda um código gerado a partir da árvore.

A função `get_min` retorna o elemento com a menor frequência da listagem passada
como parâmetro.

A função `assemble_tree` utiliza uma listagem de nós e a função `get_min` para gerar
a árvore de Huffman, retornando um ponteiro para a raiz da árvore.

A função `gen_code` recebe uma árvore e uma lista vazia de códigos, e então
preenche esta lista com os códigos encontrados na árvore. O índice de uma
posição na lista indica o valor original, e o conteúdo dessa posição indica
o código.

A função `write_tree` escreve uma lista no arquivo da seguinte forma:
* Um nó não-folha é representado como um valor 0.
* Um nó folha é representado como um valor 1 seguido pelo valor original.
* O final da árvore é representado por um 2.
* Ao final da árvore, vem o deslocamento necessário para compensar a
  desigualdade entre o número de bits e bytes.
A árvore é lida em pré-ordem e os dados são escritos no arquivo, sem as 
frequências.

A função `read_tree` realiza a operação inversa da operação `write_tree`, retornando 
um ponteiro para uma árvore de Huffman (sem as frequências).

Na função `main`, o comando é interpretado, os arquivos são abertos, as estruturas 
principais são inicializadas e há a escolha entre compressão ou extração.
Na compressão, a árvore é escrita no arquivo e o conteúdo é escrito utilizando 
um byte como buffer, que quando cheio, é escrito no arquivo.
A extração é feita lendo o arquivo bit a bit para percorrer a árvore de Huffman 
(utilizando um byte em conjunto com máscaras binárias). Cada vez que uma folha é 
encontrada, o valor desta folha é escrito no arquivo de saída.

Não há gerenciamento de memória, pois as etapas de compactação e
extração são realizadas em execuções do programa, sendo que assim que ele
termina, o SO fica encarregado de liberar a memória. Além disso, a quantidade
alocada de memória é pequena.

Para garantir a integridade do processo de compactação/extração, foi utilizada a
ferramenta de hashing sha1sum, que retorna um número que representa o arquivo.
Todos os nossos testes (inclusive os que não estão mencionados abaixo) geraram 
arquivos descompactados compatíveis com os arquivos originais.


# Testes
## Teste 1
Arquivo: JaneAusten_PrideAndPrejudice.txt

Descrição: versão integral do livro Pride and Prejudice, de Jane Austen, em 
formato de texto puro, obtido do site http://www.gutenberg.org/

Resultado:
```
    Input file size: 704139 bytes
    Output file size: 396916 bytes
    Compression rate: 0.44
    
    SHA1SUM do arquivo original:      0aa76526d7cb1ce8222ea1ea4b981aca
    SHA1SUM do arquivo descompactado: 0aa76526d7cb1ce8222ea1ea4b981aca
```    

Comentário: Pode-se notar que o tamanho caiu consideravelmente, já que o texto é
longo e utiliza um conjunto de caracteres pequeno (basicamente, os da tabela 
ASCII utilizados na língua inglesa).

## Teste 2
Arquivo: hf.c

Descrição: código fonte do programa

Resultado:
```
    Input file size: 7307 bytes
    Output file size: 4431 bytes
    Compression rate: 0.39

    SHA1SUM do arquivo original:      f4321586b894149aca2d5eb9dece266e
    SHA1SUM do arquivo descompactado: f4321586b894149aca2d5eb9dece266e
```    

Comentário: A taxa de compressão foi boa.

## Teste 3
Arquivo: hf

Descrição executável (Linux, x86) do programa

Resultado:
```
    Input file size: 11800 bytes
    Output file size: 7191 bytes
    Compression rate: 0.39
    
    SHA1SUM do arquivo original:      1aa9926bfa13392c59bdc3db8178a824
    SHA1SUM do arquivo descompactado: 1aa9926bfa13392c59bdc3db8178a824
```

Comentário: Curiosamente, o resultado da compressão do código compilado foi 
bastante semelhante ao do código fonte.

## Teste 4
Arquivo: JaneAusten_PrideAndPrejudice.zip

Descrição: versão comprimida do arquivo do Teste 1.

Resultado:
```
    Input file size: 256672 bytes
    Output file size: 257441 bytes
    Compression rate: -0.00
    
    SHA1SUM do arquivo original:      d87a77113e3092df9b7161c6acb2f75b
    SHA1SUM do arquivo descompactado: d87a77113e3092df9b7161c6acb2f75b
```

Comentário: A compressão não produziu resultados satisfatórios (de fato, houve 
uma leve perda não evidenciada pelo número de casas decimais). Isto pode ser 
explicado pela maior eficiência do algoritmo utilizado para compactar em ZIP em 
relação ao algoritmo utilizado (Huffman).

## Teste 5
Arquivo: ArquivoPequeno.txt

Descrição: arquivo de texto com pouco conteúdo.

Resultado:
```
    Input file size: 5 bytes
    Output file size: 15 bytes
    Compression rate: -2.00
    
    SHA1SUM do arquivo original:      caa2458a1e877377b7f94bf545f5c31b
    SHA1SUM do arquivo descompactado: caa2458a1e877377b7f94bf545f5c31b
```    

Comentário: conforme esperado, a compressão de um arquivo pequeno gera aumento 
do tamanho, devido ao espaço adicional necessário para armazenar no arquivo a 
árvore de Huffman utilizada.
