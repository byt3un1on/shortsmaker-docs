Diagrama de Casos de Uso,
Baseado na ideia inicial do projeto, onde o sistema recebe um tema e descrição, utiliza um LLM para decidir qual ferramenta usar, e executa a ferramenta correspondente.

Diagrama,
```mermaid
flowchart LR
    Usuario([Usuário])
    LLM([LLM / AI Engine])

    subgraph Sistema
        UC1["Enviar Tema e Descrição"]
        UC2["Analisar Contexto e<br/>Decidir Ferramenta"]
        UC3["Coletar Música<br/>NoCopyright"]
        UC4["Gerar Roteiro"]
        UC5["Gerar Vídeo"]
        UC6["Extrair Texto<br/>VTT"]
    end

    Usuario --> UC1
    UC1 --> UC2
    LLM --> UC2

    UC2 --> UC3
    UC2 --> UC4
    UC4 --> UC5
    UC5 -. extensão opcional .-> UC6
```

Este diagrama representa os casos de uso principais, mostrando as interações entre o usuário e o sistema, incluindo as decisões e execuções das ferramentas.