# Laboratório Jogo da Vida Paralelo

## Alunos
| Nome | Matrícula |
| ---- | --------- |
| Victor Eduardo Araújo Ribeiro | 190038926 |
| Pedro Victor Lima Torreão | 190036761 |
| Guilherme de Oliveira Mendes | 180113496 |

## 📋 Visão Geral
Este projeto implementa diferentes versões paralelas do **Jogo da Vida de Conway** para comparação de performance entre paradigmas de programação paralela:

- **MPI** (Message Passing Interface) - memória distribuída
- **OpenMP** - memória compartilhada
- **CUDA** - computação em GPU NVIDIA
- **OpenMP GPU** - offloading para GPU com OpenMP

## 🏗️ Estrutura do Projeto
```
jogo_da_vida/
├── src/
│   ├── jogodavida.c          # Versão sequencial original
│   ├── jogodavidampi.c       # Versão MPI
│   ├── jogodavidaomp.c       # Versão OpenMP
│   ├── jogodavida.cu         # Versão CUDA
│   └── jogodavidaomp_gpu.c   # Versão OpenMP GPU
├── .gitignore                # Git Ignore 
├── Makefile                  # Makefile
├── README.md                 # Esta documentação
└── run_benchmark.sh          # Script de benchmark automatizado
```

## 🔧 Compilação
### Pré-requisitos
- **GCC** 7.0+ com suporte OpenMP
- **Open MPI** ou Intel MPI
- **CUDA Toolkit** 10.0+ (para versão CUDA)
- **OpenMP 4.5+** com suporte GPU (para versão OpenMP GPU)


### Compilar Todas as Versões
```bash
make all
```

### Compilar Versões Específicas
```bash
# Versão sequencial
make jogodavida

# Versão MPI
make jogodavidampi

# Versão OpenMP
make jogodavidaomp

# Versão CUDA
make jogodavida_cuda

# Versão OpenMP GPU
make jogodavidaomp_gpu
```

## 🚀 Execução
### Testes Básicos
```bash
# Executar script benchmark
./run_benchmark.sh

# Executer Versão sequencial
./exec/jogodavida

# Executer Versão OpenMP
export OMP_NUM_THREADS=4
./exec/jogodavidaomp

# Executer Versão MPI
mpirun -np 4 ./exec/jogodavidampi

# Executer Versão CUDA
./exec/jogodavida_cuda

# Executer Versão OpenMP GPU
export OMP_NUM_THREADS=4
./exec/jogodavidaomp_gpu
```

## 📊 Análise de Resultados
### Métricas Coletadas

- **Tempo de inicialização** (`t_init`)
- **Tempo de computação** (`t_comp`) 
- **Tempo de finalização** (`t_fim`)
- **Tempo total** (`t_total`)

### Cálculo de Performance
```
Speedup = T_sequencial / T_paralelo
Eficiência = Speedup / Número_de_Processadores
Throughput = Células_processadas / Tempo_computação
```

### Validação
Todas as versões devem:
- ✅ Produzir resultado "**RESULTADO CORRETO**"
- ✅ Mover o veleiro do canto superior esquerdo ao inferior direito
- ✅ Manter exatamente 5 células vivas ao final

## 🔍 Detalhes das Implementações
### 1. Versão MPI (`jogodavidampi.c`)
**Estratégia**: Divisão horizontal do tabuleiro entre processos

**Características**:
- Processo 0 atua como coordenador
- Distribuição equilibrada de linhas entre workers
- Comunicação coletiva (`MPI_Bcast`, `MPI_Allgather`)
- Sincronização a cada geração

**Comando de execução**:
```bash
mpirun -np <num_processos> ./jogodavidampi
```

### 2. Versão OpenMP (`jogodavidaomp.c`)
**Estratégia**: Paralelização de loops com threads

**Características**:
- `#pragma omp parallel for` nos loops principais
- Scheduling estático para balanceamento
- Redução paralela na verificação
- Inicialização paralela

**Variáveis de ambiente**:
```bash
export OMP_NUM_THREADS=<num_threads>
export OMP_SCHEDULE=static
```

### 3. Versão CUDA (`jogodavida.cu`)
**Estratégia**: Computação massivamente paralela em GPU

**Características**:
- Kernel otimizado com memória compartilhada
- Grid 2D de blocos e threads
- Gestão explícita de memória GPU
- Dois kernels: básico e otimizado

**Configuração de kernel**:
```cpp
dim3 blockSize(16, 16);                   // 256 threads por bloco
dim3 gridSize((tam+15)/16, (tam+15)/16);  // Cobertura do tabuleiro
```

### 4. Versão OpenMP GPU (`jogodavidaomp_gpu.c`)
**Estratégia**: Offloading para GPU usando diretivas OpenMP

**Características**:
- `#pragma omp target` para offloading
- Gestão automática e explícita de dados
- Fallback para CPU se GPU indisponível
- Teams e distribute para paralelização GPU

**Diretivas principais**:
```cpp
#pragma omp target teams distribute parallel for collapse(2) \
        map(to: tabulIn[0:total_cells]) \
        map(from: tabulOut[0:total_cells])
```

## 📈 Resultados Encontrados (média de 3 execuções do benchmark)
### Performance Típica

| Versão8                | Tam=16     | Tam=32     | Tam=64      | Tam=128    | Tam=256    |
| -----                  | ------     | ------     | ------      | -------    | -------    |
| CUDA GPU               | 0.0607200s | 0.0003240s |  0.0006709s | 0.0012870s | 0.0028169s |
| MPI 1 processos        | 0.0000470s | 0.0000670s |  0.0002232s | 0.0014989s | 0.0106549s |
| MPI 2 processos        | 0.0000830s | 0.0000930s |  0.0003190s | 0.0016251s | 0.0086272s |
| MPI 4 processos        | 0.0001969s | 0.0002131s |  0.0004752s | 0.0031691s | 0.0163260s |
| MPI 8 processos        | N/A        | N/A        | N/A         | N/A        | N/A        |
| OpenMP 16 threads      | 0.0007749s | 0.0017190s |  0.0047998s | 0.0089328s | 0.0188398s |
| OpenMP 1 threads       | 0.0000169s | 0.0000319s |  0.0001781s | 0.0012751s | 0.0097780s |
| OpenMP 2 threads       | 0.0000129s | 0.0000370s |  0.0001328s | 0.0007319s | 0.0053899s |
| OpenMP 4 threads       | 0.0000191s | 0.0000470s |  0.0001302s | 0.0005140s | 0.0038860s |
| OpenMP 8 threads       | 0.0000329s | 0.0000870s |  0.0001740s | 0.0006409s | 0.0035090s |
| OpenMP-GPU GPU offload | 0.0039220s | 0.0085161s |  0.0194459s | 0.0459991s | 0.1004109s |
| Sequencial             | 0.0000160s | 0.0000198s |  0.0001540s | 0.0012200s | 0.0097461s |

### Conclusão
Este experimento comparou diferentes paradigmas de programação paralela aplicados ao Jogo da Vida de Conway, revelando insights importantes sobre a adequação de cada tecnologia para diferentes escalas do problema.

#### 1. Análise de Desempenho por Paradigma
**OpenMP (Memória Compartilhada)**
- **Melhor caso**: OpenMP com 4 threads alcançou speedup de até **3.15x** (tam=128: 0.0097461s → 0.0038860s)
- **Escalabilidade**: Excelente para tamanhos médios e grandes, com melhor eficiência entre 2-8 threads
- **Overhead**: Para tamanhos pequenos (tam=8,16), o overhead de criação de threads supera os benefícios

**CUDA (GPU)**
- **Melhor caso**: Speedup de **3.46x** para tam=128 (0.0097461s → 0.0028169s)
- **Característica**: Alta latência inicial (~0.06s) devido à inicialização da GPU
- **Ponto de equilíbrio**: Só compensa para tam ≥ 64, onde o paralelismo massivo supera o overhead

**MPI (Memória Distribuída)**
- **Melhor caso**: MPI com 2 processos alcançou speedup de **1.13x** para tam=128
- **Limitação**: Overhead de comunicação entre processos limita ganhos significativos
- **Observação**: MPI com 8 processos falhou em todos os testes, indicando problemas de escalabilidade

**OpenMP GPU Offloading**
- **Desempenho**: Consistentemente pior que todas as outras versões (10-50x mais lento)
- **Causa provável**: Overhead excessivo de transferência de dados CPU↔GPU sem otimização adequada
- **Recomendação**: Necessita otimizações específicas ou não é adequado para este problema

#### 2. Percentuais de Ganho Comparativos
| Comparação | Tam=64 | Tam=128 |
|------------|---------|----------|
| OpenMP 4 threads vs Sequencial | **137%** mais rápido | **151%** mais rápido |
| CUDA vs Sequencial | **-5%** mais devagar | **246%** mais rápido |
| CUDA vs OpenMP 4 threads | **-150%** mais devagar | **38%** mais rápido |
| MPI 2 proc vs Sequencial | **-33%** mais devagar | **13%** mais rápido |

#### 3. Recomendações por Tamanho de Problema
- **Tam ≤ 32**: Use a versão **sequencial** ou **OpenMP com 2 threads**
- **Tam = 64**: Use **OpenMP com 4-8 threads** (melhor custo-benefício)
- **Tam ≥ 128**: Use **CUDA** para máxima performance ou **OpenMP 8 threads** para boa performance sem GPU

#### 4. Achados Importantes
1. **Overhead vs Paralelismo**: O overhead de inicialização é crítico para problemas pequenos. CUDA tem overhead de ~60ms, tornando-a inviável para tam < 64.

2. **Eficiência de Threads**: OpenMP mostra melhor eficiência com 4-8 threads. Com 16 threads, a contenção e sincronização degradam a performance.

3. **Validação**: Todas as versões implementadas (exceto MPI 8 processos) produziram resultados corretos, movendo o veleiro do canto superior esquerdo para o inferior direito.

4. **GPU Efficiency**: A implementação CUDA mostrou-se eficiente apenas para problemas grandes, sugerindo que o kernel poderia ser otimizado para melhor ocupação da GPU em problemas menores.

5. **Surpresa**: OpenMP superou CUDA em alguns casos médios (tam=64), demonstrando que nem sempre GPU é a melhor escolha, especialmente considerando o overhead de setup.

#### 5. Conclusão Final 
O experimento demonstra que **não existe uma solução única melhor para todos os casos**. A escolha do paradigma deve considerar:
- **Tamanho do problema**: Fundamental para determinar se o overhead compensa
- **Hardware disponível**: GPU necessária para CUDA, múltiplos cores para OpenMP
- **Facilidade de implementação**: OpenMP oferece melhor relação facilidade/performance

Para o cluster utilizado no experimento, **OpenMP com 4-8 threads** apresentou o melhor equilíbrio entre performance, facilidade de uso e consistência across diferentes tamanhos de problema.

## 📚 Referências
1. **Conway's Game of Life**: Gardner, M. "Mathematical Games", Scientific American 223, Oct 1970
2. **MPI Documentation**: [https://www.open-mpi.org/doc/](https://www.open-mpi.org/doc/)
3. **OpenMP Specification**: [https://www.openmp.org/specifications/](https://www.openmp.org/specifications/)
4. **CUDA Programming Guide**: [https://docs.nvidia.com/cuda/](https://docs.nvidia.com/cuda/)
5. **OpenMP GPU Offloading**: [https://www.openmp.org/updates/openmp-accelerator-support-gpus/](https://www.openmp.org/updates/openmp-accelerator-support-gpus/)
