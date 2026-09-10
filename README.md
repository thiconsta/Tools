# Ferramentas para identificação e recuperação de hashes

Este documento apresenta três recursos úteis em laboratórios de cibersegurança, CTFs, auditorias autorizadas e estudos sobre armazenamento de senhas: **CrackStation**, **Hashes.com** e **hashID**.

> [!IMPORTANT]
> Use estas ferramentas somente com hashes próprios, ambientes de laboratório ou sistemas para os quais você tenha autorização explícita. Não envie hashes corporativos, credenciais reais ou dados sensíveis a serviços públicos.

## Conceito importante

Um hash é o resultado de uma função unidirecional. Portanto, ele não é tecnicamente “descriptografado”. As ferramentas de recuperação normalmente:

- calculam hashes de palavras candidatas e comparam os resultados;
- consultam tabelas ou bancos de dados previamente calculados;
- utilizam listas de palavras, regras e força bruta.

Quando uma correspondência é encontrada, a ferramenta apresenta o texto original conhecido. Isso não significa que o algoritmo de hash foi revertido matematicamente.

## Resumo das ferramentas

| Ferramenta | Função principal | Execução | Melhor uso |
| --- | --- | --- | --- |
| [CrackStation](https://crackstation.net/) | Procurar o texto correspondente a hashes sem salt em tabelas pré-calculadas | Online | Consultas rápidas em CTFs e laboratórios |
| [Hashes.com](https://hashes.com/en/decrypt/hash) | Pesquisar hashes em uma base de resultados previamente recuperados | Online | Consultar diferentes formatos e hashes já conhecidos |
| [hashID](https://pypi.org/project/hashID/) | Identificar possíveis algoritmos ou formatos de um hash | Local, via terminal | Descobrir qual tipo de hash deve ser analisado |

## 1. CrackStation

O **CrackStation** é um serviço gratuito de consulta de hashes. Ele compara o valor informado com grandes tabelas pré-calculadas que relacionam senhas e seus respectivos hashes.

### O que ele faz

- aceita até 20 hashes por consulta, um por linha;
- procura correspondências em grandes listas de palavras e senhas conhecidas;
- retorna o texto correspondente quando o hash já existe em sua base;
- funciona principalmente com hashes **sem salt**.

Entre os formatos informados como suportados estão LM, NTLM, MD2, MD4, MD5, SHA-1, SHA-224, SHA-256, SHA-384, SHA-512, RIPEMD-160, Whirlpool e MySQL 4.1+.

### Como usar

1. Acesse [crackstation.net](https://crackstation.net/).
2. Cole um ou mais hashes no campo indicado, utilizando uma linha para cada valor.
3. Resolva o CAPTCHA.
4. Inicie a consulta.
5. Verifique se a base encontrou uma correspondência.

### Limitações

- não identifica ou recupera toda senha existente;
- depende de o texto correspondente já estar contemplado nas tabelas;
- hashes com salt não são adequados para esse tipo de tabela pré-calculada;
- senhas longas, fortes e inéditas tendem a não ser encontradas.

## 2. Hashes.com

O **Hashes.com** oferece uma busca online em uma base de hashes previamente recuperados. Apesar de a página utilizar a palavra “decrypt”, o serviço informa que não quebra cada hash em tempo real: ele procura o valor em seu banco de resultados acumulados.

### O que ele faz

- recebe até 25 entradas por consulta, separadas por linha;
- aceita a estrutura `hash:salt` em formatos compatíveis;
- pode exibir o algoritmo associado aos resultados encontrados;
- pesquisa formatos como MD5, SHA-1, MySQL, NTLM, SHA-256, SHA-512, bcrypt, WordPress e outros.

### Como usar

1. Acesse a página [Decrypt Hashes](https://hashes.com/en/decrypt/hash).
2. Insira os hashes, um por linha.
3. Quando aplicável, informe o salt no formato `hash:salt`.
4. Se desejar, habilite a opção para mostrar o algoritmo encontrado.
5. Resolva o CAPTCHA e envie a consulta.

### Limitações

- o resultado depende de uma correspondência já presente na base;
- suporte a um formato não garante que determinado hash será encontrado;
- algoritmos lentos e com salt, como bcrypt, são projetados para dificultar ataques e podem não produzir resultado;
- o envio ocorre para um serviço externo, portanto não deve envolver dados sigilosos.

## 3. hashID

O **hashID** é uma ferramenta de linha de comando escrita em Python. Diferentemente dos dois serviços anteriores, ela não recupera senhas: analisa a estrutura do valor e indica quais algoritmos ou formatos podem tê-lo produzido.

A identificação é baseada principalmente em padrões e expressões regulares. Por isso, o resultado representa **possibilidades**, não uma confirmação absoluta. Formatos com o mesmo tamanho e estrutura podem gerar múltiplas sugestões.

### Recursos

- reconhece mais de 220 tipos de hash;
- analisa um valor isolado, arquivos ou arquivos em um diretório;
- pode mostrar o modo correspondente do Hashcat;
- pode mostrar o formato correspondente do John the Ripper;
- funciona localmente, sem a necessidade de enviar o hash a uma página web.

### Instalação

```bash
python3 -m pip install hashid
```

> [!NOTE]
> A versão mais recente publicada no PyPI é a 3.1.4, de março de 2015. Por ser um projeto antigo, pode não reconhecer formatos mais recentes.

### Exemplos de uso

Identificar um hash diretamente:

```bash
hashid '5f4dcc3b5aa765d61d8327deb882cf99'
```

Analisar valores armazenados em um arquivo:

```bash
hashid hashes.txt
```

Exibir também os modos compatíveis com Hashcat:

```bash
hashid -m '5f4dcc3b5aa765d61d8327deb882cf99'
```

Exibir os formatos correspondentes do John the Ripper:

```bash
hashid -j '5f4dcc3b5aa765d61d8327deb882cf99'
```

Listar possibilidades adicionais, inclusive formatos com salt:

```bash
hashid -e 'HASH_A_SER_ANALISADO'
```

Em sistemas Linux e Unix, utilize aspas simples ao passar o valor diretamente no terminal. Isso evita que caracteres especiais sejam interpretados pelo shell.

## Fluxo recomendado em um laboratório

1. Observe o tamanho, os caracteres e a estrutura do hash.
2. Execute o `hashID` para obter uma lista de formatos prováveis.
3. Use informações do contexto — sistema operacional, aplicação e local de coleta — para reduzir as possibilidades.
4. Em um ambiente autorizado, consulte o CrackStation ou o Hashes.com para verificar se o valor já é conhecido.
5. Se não houver correspondência, utilize uma ferramenta local apropriada, como Hashcat ou John the Ripper, respeitando o escopo e as regras do laboratório.

## Comparação prática

- **Preciso descobrir o provável formato:** use o hashID.
- **Tenho um hash sem salt e quero uma consulta rápida:** tente o CrackStation.
- **Quero consultar uma base com vários formatos ou uma entrada `hash:salt`:** tente o Hashes.com.
- **O dado é real ou sensível:** mantenha a análise local e não envie o hash a sites públicos.

## Referências

- [CrackStation — Free Password Hash Cracker](https://crackstation.net/)
- [Hashes.com — Hash Lookup Service](https://hashes.com/en/decrypt/hash)
- [hashID no Python Package Index](https://pypi.org/project/hashID/)

## Aviso legal e ético

Este conteúdo possui finalidade exclusivamente educacional. A análise ou tentativa de recuperação de credenciais sem autorização pode violar leis, contratos e políticas de segurança. Trabalhe apenas dentro de um escopo formalmente autorizado.
