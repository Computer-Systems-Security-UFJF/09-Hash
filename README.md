# 09-Hash

- **SHA-256 passo a passo**, com o exemplo `"red block blue"` do vídeo
  [SHA-256 | RedBlockBlue](https://www.youtube.com/watch?v=orIgy2MjqrA) (abaixo).
- **Pesquisa sobre os principais algoritmos de hash**: [`relatorio.txt`](relatorio.txt).
- **Hash simples** (XOR bloco a bloco e XOR com rotação): [`hash_simples.ipynb`](hash_simples.ipynb).

## SHA-256 passo a passo

Entrada de qualquer tamanho, saída sempre de 256 bits (64 hex).

1. **ASCII -> bits.** `red block blue` tem 14 caracteres = 112 bits
   (`r` = `01110010`, `e` = `01100101`, ...).

2. **Padding até 512 bits.** Acrescenta-se um bit `1`, depois zeros até 448 bits,
   e nos últimos 64 bits o tamanho original (`112` = `0x70`):

   ```
   [ 112 bits da mensagem ][ 1 ][ 335 zeros ][ 112 em 64 bits ]  = 512 bits
   ```

3. **16 -> 64 palavras.** O bloco vira 16 palavras de 32 bits (`W[0..15]`) e são geradas
   mais 48 com rotações e XOR:

   ```
   W[t] = σ1(W[t-2]) + W[t-7] + σ0(W[t-15]) + W[t-16]      (mod 2^32)
   σ0(x) = ROTR7(x) ^ ROTR18(x) ^ SHR3(x)
   σ1(x) = ROTR17(x) ^ ROTR19(x) ^ SHR10(x)
   ```

4. **Constantes.** `H0..H7` = parte fracionária da raiz quadrada dos 8 primeiros primos
   (`H0 = 0x6a09e667`); `K[0..63]` = parte fracionária da raiz cúbica dos 64 primeiros primos.

5. **64 rodadas.** Copiam-se `H0..H7` em `a..h` e, para cada `t`:

   ```
   T1 = h + Σ1(e) + Ch(e,f,g) + K[t] + W[t]
   T2 = Σ0(a) + Maj(a,b,c)
   h=g  g=f  f=e  e=d+T1  d=c  c=b  b=a  a=T1+T2
   ```

   onde `Ch` escolhe bits de `f` ou `g` conforme `e`, `Maj` é a maioria de `a,b,c`,
   e `Σ0`, `Σ1` são XOR de três rotações.

6. **Soma e saída.** `H[i] += a..h` e concatenam-se os oito valores:

   ```
   SHA-256("red block blue") = b0bf36f41c93c6597d2806a34b81cb0703244125d06d33f44a08834b71f30164
   ```

## Hash simples (notebook)

Dois hashes de 64 bits em Python puro, com o mesmo esqueleto do SHA (padding -> blocos -> misturar):

- `hash_xor`: `estado ^= bloco`. Trocar a ordem dos blocos não muda o hash e um bit da
  entrada altera um bit da saída.
- `hash_xor_rot`: entre os blocos aplica rotação à esquerda e soma com uma constante.
  Quebra a comutatividade e espalha os bits (cerca de metade muda ao trocar uma letra).

O notebook mostra o efeito avalanche, uma colisão trivial do `hash_xor` que o `hash_xor_rot`
evita, e compara com `hashlib.sha256`.

## Como rodar

```bash
uv sync
uv run jupyter nbconvert --to notebook --execute --inplace hash_simples.ipynb
```

Ou abra o notebook e selecione o kernel de `.venv/bin/python`.
