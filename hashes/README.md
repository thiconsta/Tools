# Ferramentas para identificação e recuperação de hashes

Este documento reúne recursos úteis em laboratórios de cibersegurança, CTFs, auditorias autorizadas e estudos sobre armazenamento de senhas. As ferramentas foram separadas por finalidade: identificação, consulta, extração, recuperação e verificação de integridade.

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
| [Hashcat](https://hashcat.net/hashcat/) | Testar candidatos contra hashes utilizando CPU, GPU ou outros dispositivos | Local | Recuperação de senhas em auditorias e CTFs |
| [John the Ripper](https://www.openwall.com/john/) | Auditar e recuperar senhas de diversos formatos | Local | Fluxo simples e suporte a arquivos convertidos por `*2john` |
| [CyberChef](https://gchq.github.io/CyberChef/) | Gerar, converter e analisar hashes por meio de receitas visuais | Navegador ou local | Testes rápidos, transformações e validação manual |
| `sha256sum` / OpenSSL | Calcular e conferir hashes de arquivos | Local | Verificação de integridade |

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

## 4. Hashcat

O **Hashcat** é uma ferramenta de recuperação de senhas otimizada para aproveitar CPU, GPU e outros dispositivos compatíveis. Ele suporta centenas de tipos de hash e diferentes estratégias de geração de candidatos.

Dois parâmetros são fundamentais:

- `-m`: seleciona o tipo de hash, chamado de **hash mode**;
- `-a`: seleciona o modo de ataque.

Alguns modos comuns são `0` para MD5, `100` para SHA-1, `1000` para NTLM, `1400` para SHA-256 e `1800` para SHA-512 Crypt. Confirme sempre o modo na documentação, pois hashes com o mesmo comprimento podem representar algoritmos diferentes.

### Exemplos básicos

Ataque de dicionário contra um hash MD5:

```bash
hashcat -m 0 -a 0 hashes.txt wordlist.txt
```

Exibir resultados já recuperados:

```bash
hashcat -m 0 hashes.txt --show
```

Executar o benchmark dos dispositivos disponíveis:

```bash
hashcat -b
```

### Modos de ataque mais utilizados

| Valor de `-a` | Tipo | Uso |
| --- | --- | --- |
| `0` | Dicionário | Testa candidatos de uma wordlist |
| `1` | Combinação | Combina palavras de duas listas |
| `3` | Máscara | Testa padrões de caracteres definidos pelo operador |
| `6` | Híbrido | Wordlist seguida por máscara |
| `7` | Híbrido | Máscara seguida por wordlist |

### Hashcat Example Hashes

A página [Example hashes](https://hashcat.net/wiki/doku.php?id=example_hashes) relaciona cada modo numérico ao nome e a um exemplo válido. Ela é especialmente útil para:

- localizar o `-m` correto;
- comparar a estrutura do hash recebido;
- conferir a posição do salt;
- testar um comando com um exemplo conhecido;
- investigar erros como `Token length exception` ou `Separator unmatched`.

> [!TIP]
> A documentação informa que erros de comprimento de linha geralmente indicam que o modo selecionado não corresponde ao formato do hash. Identificação automática ajuda, mas o contexto de origem continua sendo essencial.

## 5. John the Ripper

O **John the Ripper** é uma ferramenta de auditoria e recuperação de senhas. A edição **Jumbo** amplia o suporte para hashes de sistemas, aplicações web, bancos de dados, capturas de rede, chaves privadas, carteiras, discos, arquivos compactados e documentos.

### Exemplos básicos

Executar um ataque de dicionário:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

Deixar que o John tente reconhecer o formato:

```bash
john hashes.txt
```

Exibir resultados recuperados:

```bash
john --show hashes.txt
```

Quando houver ambiguidade, informe o formato explicitamente:

```bash
john --format=raw-md5 --wordlist=wordlist.txt hashes.txt
```

## 6. Utilitários `*2john`

Muitos alvos não entregam um hash diretamente. Os utilitários incluídos no John the Ripper Jumbo extraem os dados necessários e os convertem para um formato que John ou Hashcat possam processar.

Exemplos comuns:

```bash
zip2john arquivo.zip > zip.hash
pdf2john documento.pdf > pdf.hash
ssh2john chave_privada > ssh.hash
keepass2john banco.kdbx > keepass.hash
```

Depois da extração:

```bash
john --wordlist=wordlist.txt zip.hash
```

O nome e a disponibilidade de cada conversor dependem da instalação. Em algumas distribuições, os scripts ficam em diretórios como `/usr/share/john/`.

## 7. CyberChef

O **CyberChef**, mantido pelo GCHQ, funciona como uma bancada visual para transformação e análise de dados. Ele permite encadear operações em uma **receita**, o que é útil para entender hashes compostos e reproduzir etapas como `SHA-1` seguido de conversão para hexadecimal.

### Usos relacionados a hashes

- gerar MD5, SHA-1, SHA-2, SHA-3 e outros digests;
- calcular o hash de texto ou arquivos;
- converter entre texto, hexadecimal e Base64;
- testar HMAC com uma chave conhecida;
- encadear várias operações para compreender formatos compostos;
- comparar o resultado com um valor fornecido pelo desafio.

A aplicação oficial processa as entradas no próprio navegador e também pode ser baixada para execução local. Mesmo assim, evite incluir dados sensíveis em URLs compartilhadas, pois receitas e entradas podem fazer parte do link.

## 8. `sha256sum`, `sha1sum` e OpenSSL

Nem todo uso de hash envolve senhas. Hashes também servem para verificar se um arquivo foi alterado durante download, cópia ou análise forense.

Calcular SHA-256 no Linux:

```bash
sha256sum arquivo.iso
```

Verificar um arquivo de checksums:

```bash
sha256sum -c SHA256SUMS
```

Calcular SHA-1:

```bash
sha1sum arquivo.bin
```

Usar OpenSSL para calcular SHA-256:

```bash
openssl dgst -sha256 arquivo.iso
```

No PowerShell, o equivalente é:

```powershell
Get-FileHash .\arquivo.iso -Algorithm SHA256
```

Para integridade, prefira SHA-256 ou superior. MD5 e SHA-1 ainda aparecem como checksums legados, mas não são adequados quando existe risco de adulteração maliciosa.

## Fluxo recomendado em um laboratório

1. Observe o tamanho, os caracteres e a estrutura do hash.
2. Execute o `hashID` para obter uma lista de formatos prováveis.
3. Compare o resultado com a página de exemplos do Hashcat.
4. Use informações do contexto — sistema operacional, aplicação e local de coleta — para reduzir as possibilidades.
5. Se o alvo for um ZIP, PDF, chave ou outro arquivo, extraia o material com o utilitário `*2john` adequado.
6. Em um ambiente autorizado, consulte o CrackStation ou o Hashes.com para verificar se o valor já é conhecido.
7. Se não houver correspondência, utilize Hashcat ou John the Ripper com uma wordlist adequada ao contexto.
8. Registre o formato, comando, wordlist, regras e resultado para que o teste possa ser reproduzido.

## Comparação prática

- **Preciso descobrir o provável formato:** use o hashID.
- **Tenho um hash sem salt e quero uma consulta rápida:** tente o CrackStation.
- **Quero consultar uma base com vários formatos ou uma entrada `hash:salt`:** tente o Hashes.com.
- **Preciso encontrar o modo correto do Hashcat:** consulte Hashcat Example Hashes.
- **Quero testar muitos candidatos usando GPU:** use Hashcat.
- **Preciso trabalhar com ZIP, PDF, SSH, KeePass ou outro arquivo:** use um conversor `*2john` e depois John ou Hashcat.
- **Quero gerar ou conferir um hash visualmente:** use CyberChef.
- **Quero verificar a integridade de um arquivo:** use `sha256sum`, OpenSSL ou `Get-FileHash`.
- **O dado é real ou sensível:** mantenha a análise local e não envie o hash a sites públicos.

## Referências

- [CrackStation — Free Password Hash Cracker](https://crackstation.net/)
- [Hashes.com — Hash Lookup Service](https://hashes.com/en/decrypt/hash)
- [hashID no Python Package Index](https://pypi.org/project/hashID/)
- [Hashcat — site oficial](https://hashcat.net/hashcat/)
- [Hashcat — Example Hashes](https://hashcat.net/wiki/doku.php?id=example_hashes)
- [John the Ripper — Openwall](https://www.openwall.com/john/)
- [CyberChef — aplicação oficial](https://gchq.github.io/CyberChef/)
- [CyberChef — código-fonte e documentação](https://github.com/gchq/CyberChef)
- [OpenSSL `dgst`](https://docs.openssl.org/3.5/man1/openssl-dgst/)
- [TryHackMe — Hashing Basics](https://tryhackme.com/room/hashingbasics)

## Aviso legal e ético

Este conteúdo possui finalidade exclusivamente educacional. A análise ou tentativa de recuperação de credenciais sem autorização pode violar leis, contratos e políticas de segurança. Trabalhe apenas dentro de um escopo formalmente autorizado.
