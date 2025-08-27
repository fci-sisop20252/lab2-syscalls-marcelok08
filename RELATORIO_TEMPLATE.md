# 📝 Relatório do Laboratório 2 - Chamadas de Sistema

---

## 1️⃣ Exercício 1a - Observação printf() vs 1b - write()

### 💻 Comandos executados:
bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write


### 🔍 Análise

*1. Quantas syscalls write() cada programa gerou?*
- ex1a_printf: 1 syscalls
- ex1b_write: 7 syscalls

*2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md*


A função printf() faz parte da biblioteca padrão e utiliza um buffer interno para armazenar os dados antes de enviá-los. O envio real para o kernel só ocorre quando esse buffer é liberado (flush), o que geralmente acontece ao encontrar um caractere de nova linha (\n) ou quando o espaço do buffer se esgota. Já a write() corresponde a uma chamada direta ao sistema, de modo que cada execução é imediatamente processada pelo kernel. Por isso, printf() costuma gerar menos syscalls do que write().


*3. Qual método é mais previsível? Por quê você acha isso?*


O write() é o mais previsível, pois cada chamada feita no programa resulta em uma syscall correspondente, sem intermediação de buffer. Já no caso do printf(), o comportamento pode variar conforme o ambiente em que o programa roda (saída no terminal, em arquivo ou redirecionamento), devido ao mecanismo de buffering da biblioteca C.


---

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: 3
- Bytes lidos: 127

### 🔧 Comando strace:
bash
strace -e openat,read,close ./ex2_leitura


### 🔍 Análise

*1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?*


Os descriptores 0, 1 e 2 já estão reservados para stdin, stdout e stderr. Assim, a primeira chamada a open() recebe o próximo descritor disponível, que nesse caso foi o 3.


*2. Como você sabe que o arquivo foi lido completamente?*


Isso acontece quando a chamada read() retorna 0, o que indica que o processo chegou ao EOF (End Of File).


*3. Por que verificar retorno de cada syscall?*


Porque chamadas de sistema podem falhar por vários motivos: arquivo inexistente, falta de permissão, espaço em disco insuficiente, entre outros. Validar o retorno evita comportamentos indesejados e facilita o tratamento adequado dos erros.


---

## 3️⃣ Exercício 3 - Contador com Loop

### 📋 Resultados (BUFFER_SIZE = 64):
- Linhas: 25 (esperado: 25)
- Caracteres: 1300
- Chamadas read(): 21
- Tempo: 0.000097 segundos

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |        82       | 0.001378  |
| 64          |        21       | 0.000494  |
| 256         |        6        | 0.000299  |
| 1024        |        7        | 0.000274  |

### 🔍 Análise

*1. Como o tamanho do buffer afeta o número de syscalls?*


Buffers maiores permitem ler mais dados de uma vez, diminuindo a quantidade de chamadas read() necessárias para percorrer todo o arquivo.


*2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre*


Não. A última leitura pode retornar uma quantidade menor de bytes, caso o arquivo não seja múltiplo exato do tamanho definido no buffer.


*3. Qual é a relação entre syscalls e performance?*


Cada syscall gera um custo de transição entre espaço do usuário e do kernel. Assim, quanto mais chamadas, maior o overhead e o tempo de execução. Usar buffers maiores reduz a quantidade de syscalls e, consequentemente, melhora o desempenho.


---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: 1364
- Operações: 6
- Tempo: 0.000205 segundos
- Throughput: 6497.71 KB/s

### ✅ Verificação:
bash
diff dados/origem.txt dados/destino.txt

Resultado: [X] Idênticos [ ] Diferentes

### 🔍 Análise

*1. Por que devemos verificar que bytes_escritos == bytes_lidos?*


Isso assegura que a cópia foi realizada de forma completa, evitando arquivos corrompidos ou incompletos.


*2. Que flags são essenciais no open() do destino?*


As flags necessárias são: O_WRONLY, O_CREAT e O_TRUNC.


*3. O número de reads e writes é igual? Por quê?*


Nem sempre. Em alguns casos, um write() pode gravar menos bytes do que foram lidos, exigindo chamadas adicionais para completar a escrita.


*4. Como você saberia se o disco ficou cheio?*


Se write() retornar -1 e errno for ENOSPC, significa que o espaço em disco se esgotou.


*5. O que acontece se esquecer de fechar os arquivos?*


Arquivos não fechados continuam ocupando recursos do sistema, podendo resultar em cópia incompleta e problemas de memória ou descritores.


---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

*1. Como as syscalls demonstram a transição usuário → kernel?*


Chamadas de sistema (syscalls) são a forma do programa em espaço de usuário requisitar serviços do kernel, permitindo acessar recursos do sistema de forma segura e controlada.

*2. Qual é o seu entendimento sobre a importância dos file descriptors?*


File descriptors funcionam como identificadores de arquivos ou recursos abertos, possibilitando ler, escrever e fechar esses recursos de maneira organizada e segura.

*3. Discorra sobre a relação entre o tamanho do buffer e performance:*


Buffers maiores permitem reduzir o número de syscalls e o overhead de transição entre usuário e kernel, melhorando o desempenho. Buffers pequenos aumentam as chamadas e tornam a operação mais lenta.


### ⚡ Comparação de Performance

bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt


*Qual foi mais rápido?* user

*Por que você acha que foi mais rápido?*


O tempo medido em "user" contabiliza apenas o processamento no espaço do usuário, sem incluir o overhead das transições para o kernel, que normalmente são mais custosas.


---

## 📤 Entrega
Certifique-se de ter:
- [X] Todos os códigos com TODOs completados
- [X] Traces salvos em traces/
- [X] Este relatório preenchido como RELATORIO.md

bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia

# Bom trabalho!
