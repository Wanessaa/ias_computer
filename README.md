# ias_computer

<!-- <Flowchart> -->

### Fluxograma do funcionamento do Computador IAS

```mermaid
---
title: Flowchart of IAS Operation
---

%% # Integrantes do projeto:
%% # - Eliane de Melo Cordeiro (GitHub: ElianeCordeiro)
%% # - Flávio Filipe França Farias (GitHub: trewq010)
%% # - Maria Eduarda Veloso Cânha (GitHub: dudacanha)
%% # - Laura Maria Farias Silva (GitHub: laura-farias-dev)
%% # - Wanessa Santana Ferreira (GitHub: Wanessaa)

flowchart TD

subgraph "Fetch Cycle"
    START((Start)):::greenClass --> B{"Is next 
    instruction 
    in IBR?"}
    B --> |"Yes

    No memory access
    required"| F("IR ← IBR(0:7)
    MAR ← IBR(8:19)")
    F --> I
    B --> |No| C("MAR ← PC")
    C --> D("MBR ← M(MAR)")
    D --> E{"Left
    instruction
    required?"}
    E --> |No| H("IR ← MBR(20:27)
    MAR ← MBR(28:39)")
    E --> |Yes| G("IBR ← MBR(20:39)
    IR ← MBR(0:7)
    MAR ← MBR(8:19)")
    H --> I("PC ← PC + 1")
end

subgraph DECODE
	DECODE_A{{"Decode instruction in IR"}}:::orangeClass
    I --> DECODE_A
    G --> DECODE_A
end
style DECODE fill:transparent,stroke:transparent

subgraph "Execution Cycle"
    DECODE_A --> |"STOR M(X, 8:19)
    (opcode: 00010010)" | STOR_MXL

     %% A instrução STOR M(X, 8:19) substitui os bits de endereço da instrução esquerda, do bit 28 ao 39, de um local de memória chamado M(X), pelos bits do 28 ao 39 do acumulador (AC). Para isso, o processo passa pelo Registrador de Memória do Barramento (MBR), pois o AC não tem acesso direto à memória. Dessa forma, o conteúdo da memória tem que ser transferido para o MBR, que pode acessar tanto a memória quanto o AC. Então, o MBR substitui os bits do 8 ao 19 pelo conteúdo dos bits do 28 ao 39 do AC. Após a modificação, o conteúdo atualizado é transferido de volta para a memória M(X), finalizando o processo de modificação do campo de endereço da instrução esquerda em M(X).

    subgraph STOR_MXL ["STOR M(X, 8:19)"]
        STOR_MXL1("MBR ← M(MAR)")
        STOR_MXL2("MBR(8:19) ← AC(28:39)")
        STOR_MXL3("M(MAR) ← MBR")

        STOR_MXL1 ---> STOR_MXL2
        STOR_MXL2 ---> STOR_MXL3
    end


    DECODE_A --> |"STOR M(X, 28:39)
    (opcode: 00010011)"|STOR_MXR 

    %% A instrução STOR M(X, 28:39) substitui os bits de endereço na instrução direita, do bit 28º ao 39º, de um local de memória chamado M(X), pelos respectivos bits do 28º ao 39º do acumulador (AC). Para isso, o processo passa pelo Registrador de Memória do Barramento (MBR), pois o AC não tem acesso direto à memória. Dessa forma, o conteúdo da memória tem que ser transferido para o MBR, que pode acessar tanto a memória quanto o AC.Então, o MBR substitui os bits do 28º ao 39º pelo conteúdo correspondente do AC. Após a modificação, o conteúdo atualizado é transferido de volta para a memória M(X), finalizando o processo de modificação do campo de endereço da instrução direita em M(X).

    subgraph STOR_MXR ["STOR M(X, 28:39)"]
        STOR_MXR1("MBR ← M(MAR)")
        STOR_MXR2("MBR(28:39) ← AC(28:39)")
        STOR_MXR3("M(MAR) ← MBR")

        STOR_MXR1 ---> STOR_MXR2
        STOR_MXR2 ---> STOR_MXR3
    end


    DECODE_A --> |"JUMP M(X, 0:19)
    (opcode: 00001101)"|JUMP_ML

   %% A instrução JUMP M(X, 0:19), ao receber o endereço de memória (MAR) do ciclo de busca, acessa a palavra de memória que contém duas intruções e armazena esse conteúdo no MBR. O endereço de memória presente na instrução localizada à esquerda desse conteúdo é lido e armazenado no contador de programa (PC). O que indica que o conteúdo apontado por esse endereço será executado no próximo ciclo de busca, independente do conteúdo do registrador IBR ou o que estava armazenado anteriormente em PC.

    subgraph JUMP_ML ["JUMP M(X, 0:19)"]
	JUMP_MXL1("MBR ← M(MAR)")
        JUMP_MXL2("MAR ← MBR(8:19)")
        JUMP_MXL3("PC ← MAR")
        JUMP_MXL4("Reset IBR")

        JUMP_MXL1 ---> JUMP_MXL2
        JUMP_MXL2 ---> JUMP_MXL3
        JUMP_MXL3 ---> JUMP_MXL4
    end


    DECODE_A --> |"JUMP M(X, 20:39)
    (opcode: 00001110)"|JUMP_MR

    %% A instrução JUMP M(X, 20:39),  ao receber o endereço de memória (MAR) do ciclo de busca, acessa a palavra de memória que contém duas intruções e armazena esse conteúdo no MBR. O endereço de memória presente na instrução localizada à direita desse conteúdo é lido e armazenado no contador de programa (PC). O que indica que o conteúdo apontado por esse endereço será executado no próximo ciclo de busca, independente do conteúdo do registrador IBR ou o que estava armazenado anteriormente em PC.

    subgraph JUMP_MR ["JUMP M(X, 20:39)"]
        JUMP_MXR1("MBR ← M(MAR)")
        JUMP_MXR2("MAR ← MBR(28:39)")
        JUMP_MXR3("PC ← MAR")
        JUMP_MXR4("Reset IBR")

        JUMP_MXR1 ---> JUMP_MXR2
        JUMP_MXR2 ---> JUMP_MXR3
        JUMP_MXR3 ---> JUMP_MXR4
    end


    DECODE_A ----> |"JUMP+ M(X, 0:19)
    (opcode: 00001111)"|JUMP+_ML

    %% A instrução JUMP+ M(X, 0:19) tem o efeito de saltar para a instrução esquerda da memória apenas se o valor contido no registrador AC for maior ou igual a zero, indicando que AC não é um número negativo. Caso contrário, se o valor em AC for negativo, o fluxo de execução continua normalmente, mantendo os mesmos valores nos registradores.

    subgraph JUMP+_ML ["JUMP+ M(X, 0:19)"]
        JUMP1_MXL1{"AC>= 0"}
        JUMP1_MXL2("MBR ← M(MAR)")
        JUMP1_MXL3("MAR ← MBR(8:19)")
        JUMP1_MXL4("PC ← MAR")
        JUMP1_MXL5("Reset IBR")
       
        JUMP1_MXL1 --> |Yes| JUMP1_MXL2
        JUMP1_MXL2 -->  JUMP1_MXL3
        JUMP1_MXL3 -->  JUMP1_MXL4
        JUMP1_MXL4 -->  JUMP1_MXL5   
    end


    DECODE_A ----> |"JUMP+ M(X, 20:39)
    (opcode: 00010000)"|JUMP+_MR

    %% A instrução JUMP+ M(X, 20:39) tem o efeito de saltar para a instrução direita da memória apenas se o valor contido no registrador AC for maior ou igual a zero, indicando que AC não é um número negativo. Caso contrário, se o valor em AC for negativo, o fluxo de execução continua normalmente, mantendo os mesmos valores nos registradores.  
      
    subgraph JUMP+_MR ["JUMP+ M(X, 20:39)"]
        JUMP1_MXR1{"AC>= 0"}
        JUMP1_MXR2("MBR ← M(MAR)")
        JUMP1_MXR3("MAR ← MBR(28:39)")
        JUMP1_MXR4("PC ← MAR")
        JUMP1_MXR5("Reset IBR")

        JUMP1_MXR1 --> |Yes| JUMP1_MXR2
        JUMP1_MXR2 -->  JUMP1_MXR3
        JUMP1_MXR3 -->  JUMP1_MXR4
        JUMP1_MXR4 -->  JUMP1_MXR5
    end
end

subgraph END_S ["End"]
    END(("Go back
    to Start")):::greenClass

    STOR_MXL --> END
    STOR_MXR  --> END
    JUMP_ML --> END
    JUMP_MR --> END
    JUMP1_MXL5 --> END
    JUMP1_MXL1 --> |No| END
    JUMP1_MXR5 --> END
    JUMP1_MXR1 --> |No| END      
end

style END_S fill:transparent,stroke:transparent

classDef greenClass fill:#008000
classDef orangeClass fill:#FF6347


```
