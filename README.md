# Tarefa 4 - Fine-tuning do SAM com ISIC 2018

Alunos: Kauan Mariani Ferreira, Pedro Henrique Coterli e Samuel Corrêa Lima

Link para o vídeo com a apresentação: https://youtu.be/y18fYMNpD8c

Este repositório contém a implementação do fine-tuning do **Segment Anything Model (SAM)** para a tarefa de segmentação de lesões de pele, utilizando o dataset do **ISIC 2018 Challenge**.

## Arquivos no repositório:

- `sam-model.ipynb` — Notebook com a implementação principal, o processo de fine-tuning e a avaliação do modelo.
- `README.md` — Este arquivo, com a descrição detalhada do projeto.
 
---

# Descrição do Experimento: Fine-Tuning do SAM

Este documento descreve a implementação e a metodologia para especializar o modelo SAM, originalmente treinado para segmentação genérica, em uma tarefa de domínio específico: a segmentação de imagens médicas de lesões de pele.

## 1. Objetivo do Experimento

O objetivo principal é adaptar o SAM para segmentar lesões de pele com alta precisão, aproveitando seus recursos pré-treinados. A estratégia consiste em congelar o codificador de imagem (um grande Vision Transformer) e treinar apenas o decodificador de máscara, tornando o processo computacionalmente eficiente e focado na tarefa de destino.

## 2. Pipeline do Modelo

### a. Dataset e Preparação dos Dados

- **Dataset**: Foi utilizado o **ISIC 2018 Challenge**, que contém imagens de lesões de pele e suas respectivas máscaras de segmentação.
- **Carregamento dos Dados**: Uma classe `Dataset` customizada foi implementada para carregar as imagens e máscaras, aplicando as transformações necessárias. O dataset foi dividido em conjuntos de treinamento e validação.

- **Data Augmentation**: Foram aplicadas técnicas de Data Augmentation nos dados, para aumentar a robustez do modelo.

### b. Arquitetura do Modelo

- **Modelo Base**: Foi utilizado o modelo **SAM com backbone ViT-Huge (`sam_vit_h_4b8939.pth`)**, pré-treinado em um vasto conjunto de dados.
- **Estratégia de Fine-Tuning**:
    - O **codificador de imagem (Image Encoder)** e o **codificador de prompt (Prompt Encoder)** foram mantidos congelados (sem atualização de pesos).
    - Apenas o **decodificador de máscara (Mask Decoder)** foi treinado. Essa abordagem permite que o modelo aprenda a interpretar os recursos extraídos pelo backbone para a tarefa específica de segmentação de lesões.

### c. Função de Perda

A função de perda total é uma combinação ponderada de duas funções, uma abordagem comum e eficaz para tarefas de segmentação:

1.  **Dice Loss**: Mede a sobreposição entre a máscara predita e a máscara real. É especialmente útil para lidar com o desequilíbrio entre pixels de primeiro e segundo plano.
2.  **Binary Cross Entropy**: Cross-Entropy Loss como função de perda.

### d. Processo de Treinamento

- **Otimizador**: `AdamW`, que é adequado para modelos baseados em transformers.
- **Loop de Treinamento**: Em cada iteração, o modelo recebe uma imagem e um "prompt" na forma de uma bounding box (caixa delimitadora) derivada da máscara real. O modelo então gera uma máscara de segmentação.
- **Cálculo do Gradiente**: A perda combinada é calculada, e os gradientes são retropropagados para atualizar apenas os pesos do decodificador de máscara.
- **Avaliação**: A performance do modelo é medida com a métrica **Intersection over Union (IoU)**, que avalia a qualidade da sobreposição entre a máscara predita e a real.
