# Header BGP (Border Gateway Protocol)

## 1. Visão Geral -

O BGP, atualmente em sua versão 4 (BGP-4), é o protocolo de roteamento de facto da internet, classificado como um exterior gateway protocol (EGP). Embora opere na Camada 7 (aplicação) do modelo OSI, utilizando o TCP como transporte (porta 179), sua função é puramente de infraestrutura de rede, gerenciando a alcançabilidade entre sistemas autônomos.

Diferente de protocolos orientados a pacotes individuais (como IP ou UDP), o BGP opera sobre um fluxo de bytes contínuo (byte stream) fornecido pelo TCP. Isso impõe uma necessidade crítica de delimitação de mensagens. É aqui que entra o jeader BGP.

O cabeçalho BGP é uma estrutura fixa de 19 bytes que precede toda e qualquer mensagem BGP. Ele atua como o envelope de controle, garantindo a sincronização do fluxo, a determinação do tamanho da mensagem e a identificação do tipo de payload que segue.

**Normatização -**
A estrutura e o comportamento do cabeçalho são definidos primariamente pela RFC 4271. Extensões e clarificações relevantes incluem:
- **RFC 4271:** Especificação base.
- **RFC 8654:** Extended Message Support for BGP (altera o comportamento do campo length).
- **RFC 2918:** Define o tipo de mensagem route-refresh.

---

**Anatomia Estrutural -**

O cabeçalho BGP é transmitido em network byte order (big endian). Isso significa que o byte mais significativo é transmitido primeiro. Para um desenvolvedor escrevendo um parser, é imperativo converter os valores de múltiplos bytes (como length) de big endian para a arquitetura do host (geralmente little endian em x86/x64) usando funções como `ntohs()`.

**Diagrama da Estrutura -**

```text
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
+                                                               +
|                                                               |
+                                                               +
|                           Marker                              |
+                                                               +
|                      (16 Octetos)                             |
+                                                               +
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Length (2 Octetos)   |  Type (1 Oct) |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

## 2. Decomposição dos Campos "Base" -

| Campo | Tamanho (Bits) | Tamanho (Bytes) | Offset (Byte) | Tipo de Dado | Descrição |
| :--- | :---: | :---: | :---: | :--- | :--- |
| **Marker** | 128 | 16 | 0 - 15 | Bitmap / Binary | Sincronização e Compatibilidade. |
| **Length** | 16 | 2 | 16 - 17 | Unsigned Integer (16-bit) | Comprimento total da mensagem (Header + Payload). |
| **Type** | 8 | 1 | 18 | Unsigned Integer (8-bit) | Código do tipo da mensagem. |

---

### 2.1 Marker -

O campo marker, que ocupa os primeiros 128 bits do cabeçalho BGP, desempenha um papel fundamental na integridade da comunicação entre roteadores. Historicamente, nas primeiras versões do protocolo (como na RFC 1105), esse espaço foi projetado para conter dados de autenticação, como assinaturas criptográficas. No entanto, com a evolução da tecnologia, a segurança passou a ser tratada de outras formas, como através da opção TCP MD5 signature ou dentro de atributos de caminho, alterando o propósito prático deste campo.

Como no mundo da engenharia de redes é quase impossível mudar a estrutura base de um protocolo sem criar um completamente novo, esses 16 bytes, que antes eram destinados para criptografia, continuaram lá. Então, atualmente, para garantir compatibilidade estrita com a RFC 4271, o campo marker deve ser preenchido, por padrão, com todos os bits em 1 (uma sequência hexadecimal completa de FF). Essa padronização é crítica devido à natureza do protocolo de transporte subjacente, como o BGP roda sobre TCP, que é um fluxo contínuo de dados sem delimitação de mensagens, o roteador precisa de uma referência clara para identificar onde uma mensagem começa e termina.

Dessa forma, o Marker atua como uma âncora lógica para sincronização. Caso um roteador perca a noção dos limites da mensagem devido a erros de processamento ou corrupção, ele varre o fluxo de entrada em busca dessa sequência de 16 bytes de 0xFF para se ressincronizar e processar a próxima mensagem válida. Em cenários modernos, especialmente em mensagens do tipo OPEN que não utilizam métodos de autenticação legados, o preenchimento com todos os bits em 1 é obrigatório, podendo resultar na recusa da conexão caso o padrão não seja respeitado.
