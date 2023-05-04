---
layout: post
title: "Como funciona a compressão de dados"
date: 2023-05-04 05:10:00 -0300
categories: programação algoritmos logica dados
---

Olá! Nesse post, vamos explorar os conceitos de compressão de dados, na faculdades de ciencia da computação estudamos que em computadores tudo é um grande "LINGUISÃO DE BITS" que apenas é traduzido por tabelação para caractere, tendo isso em mente vamos entender como funciona o pensamento sobre compressão de dados.

# 1. O que é a compressão de dados

Comprimir é o ato de torna algo menor (espremer) ou (remover). Existem muitos tipo de compressão de dados, aquelas que perdem dados e não tem como reculperar e aquelas que perdem e tem como reculperar.

# Vamos para a implementação

Criei um diretorio *teste* no meu wsl ubuntu 22 para fazer esse teste.

```
mkdir teste
cd teste
```

Vou criar um arquivo de texto chamado *t.txt*

*t.txt*
```
aaa b c d e   f
```

vamo rodar um xxd nesse arquivo

```shell
italo@italo:~/teste$ xxd -b t.txt
00000000: 01100001 01100001 01100001 00100000 01100010 00100000  aaa b
00000006: 01100011 00100000 01100100 00100000 01100101 00100000  c d e
0000000c: 00100000 00100000 01100110 00001010
```

Aqui vemos que os primeiros 3 bytes se repetem no endereço offset 00000000 ``01100001 01100001 01100001`` eu sei que ja ta na cara do porque esse byte repete mais vamos ser um pouco mais hardcore e vamos fingir que não sabemos. Para descobrir vou criar um script python para isso.

```py
binario = '01100001'
decimal = int(binario, 2)
ascii_char = chr(decimal)

print(ascii_char)
```

Agora temos a total certeza que `01100001` e igual a `a` em ascii, sabendo disso se entra o conceito mais basico de compressão sem perda de dados. Se nos conseguirmos passar simplificar esse dado para apenas um trecho que represente 3 dele vamos ter um arquivo menor correto ? Sim
embora não seja gritante a diferença, esse é o algoritmo de (RLE) Run Length Encoding. Vamos implementa-lo em python agora.

```py
def compress(file_path):
    with open(file_path, 'rb') as file:
        data = file.read()
    
    compressed_data = []
    count = 1

    for i in range(1, len(data)):
        if data[i] == data[i - 1]:
            count += 1
        else:
            compressed_data.append((count, data[i - 1]))
            count = 1
            
    compressed_data.append((count, data[-1]))

    return compressed_data


def decompress(compressed_data, output_file_path):
    decompressed_data = []

    for count, byte in compressed_data:
        decompressed_data.extend([byte] * count)

    with open(output_file_path, 'wb') as output_file:
        output_file.write(bytearray(decompressed_data))

        
        
def main():
    input_file_path = 't.txt'
    compressed_file_path = 't_des.txt'

    compressed_data = compress(input_file_path)
    print(f'Dados comprimidos: {compressed_data}')

    decompress(compressed_data, compressed_file_path)
    print(f'Arquivo descomprimido salvo em: {compressed_file_path}')

if __name__ == '__main__':
    main()

```

```shell
italo@italo:~/teste$ python3 main.py
Dados comprimidos: [(3, 97), (1, 32), (1, 98), (1, 32), (1, 99), (1, 32), (1, 100), (1, 32), (1, 101), (3, 32), (1, 102), (1, 10)]
Arquivo descomprimido salvo em: t_des.txt
```
Note a saida, vemos que isso e um array de tuplas sendo o primeiro indice a quantidade de repetição e o segundo o decimal em ascii. E parabens vc fez seu primeiro algoritmo de compressão de dados.