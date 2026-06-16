# Análise 03 — Multivibradores com AmpOp

**Curso:** Tecnologia em Eletrônica Industrial / Engenharia Eletrônica  
**Unidade:** Eletrônica III  
**Professor:** Luis Carlos Martinhago Schlichting  

---

## 1. Objetivo

Analisar, simular e montar circuitos multivibradores baseados em amplificadores operacionais, determinando os limiares de transição, equações de frequência e o comportamento das saídas em função da tensão de controle Vco.

---

## 2. Circuito — Questão 1

> 📷 **[INSERIR AQUI: Imagem do esquemático do Proteus — Figura 1]**

### 2.1 Parâmetros do circuito

| Componente | Valor    | Função                                      |
|------------|----------|---------------------------------------------|
| R1         | 100 kΩ   | Converte Vco em corrente para o integrador  |
| R2         | 51 kΩ    | Referência da entrada (+) de U1:B           |
| R3         | 51 kΩ    | Divisor de histerese — comparador U1:A      |
| R4         | 51 kΩ    | Divisor da entrada (+) de U1:B              |
| R5         | 10 kΩ    | Limita corrente no reset do capacitor       |
| R6         | 100 kΩ   | Divisor de histerese — comparador U1:A      |
| R7         | 50 kΩ    | Limita corrente de base do transistor Q1    |
| C1         | 0,05 µF  | Capacitor integrador — gera a rampa         |
| Q1         | BC547    | Chave de reset de C1 (NPN)                  |
| SW1        | SW-SPST  | Habilita/desabilita o reset manual          |
| V1         | 10 V     | Alimentação de referência do integrador     |
| V2         | 2,5 V    | Tensão de referência central do comparador  |
| Vco        | 6 V      | Tensão de controle (constante — Questão 1)  |
| U1:A, U1:B | LM324   | Amplificadores operacionais                 |

---

## 3. Análise de Funcionamento

### 3.1 Diagrama de blocos

O circuito é composto por **quatro blocos funcionais** em malha fechada:

```
Vco ──► [ Bloco 1: Entrada + R1 ] ──► [ Bloco 2: Integrador U1:B ] ──► Saída 2 (rampa)
                                               │                              │
                                               │◄── Reset (Q1 satura) ◄──────┤
                                                                              ▼
                                         [ Bloco 4: Chave Q1 ] ◄─── [ Bloco 3: Comparador U1:A ] ──► Saída 1 (quadrada)
```

### 3.2 Função de cada bloco e componente

#### Bloco 1 — Entrada Vco (R1, R4)

**R1 (100 kΩ)** converte a tensão Vco em uma corrente constante que alimenta o capacitor C1 do integrador. Quanto maior Vco, maior a corrente e mais rápida a rampa — o que aumenta diretamente a frequência de oscilação.

**R4 (51 kΩ)** faz parte do divisor resistivo que fixa o potencial na entrada não-inversora (+) de U1:B, determinando o ponto de equilíbrio do integrador.

#### Bloco 2 — Integrador U1:B (C1, R2, V1)

U1:B está configurado como **integrador inversor**. O capacitor **C1 (0,05 µF)** no caminho de realimentação acumula a carga proveniente de R1 e produz uma **rampa linear** na Saída 2. A equação da tensão de saída do integrador é:

$$V_{out}(t) = -\frac{1}{R_1 \cdot C_1} \int V_{co} \, dt = -\frac{V_{co}}{R_1 C_1} \cdot t$$

**V1 (10 V)** e **R2 (51 kΩ)** fixam o potencial na entrada (+) de U1:B, garantindo operação no ponto de trabalho correto com alimentação única.

#### Bloco 3 — Comparador com Histerese U1:A (R3, R6, V2)

U1:A atua como **Schmitt Trigger** (comparador com histerese). Ele compara a rampa da Saída 2 com o limiar definido pelo divisor resistivo composto por **R3 (51 kΩ)** e **R6 (100 kΩ)** a partir de **V2 (2,5 V)** e da própria saída de U1:A.

- Quando a rampa atinge **V_TH** → Saída 1 comuta para nível baixo
- Quando a rampa cai até **V_TL** → Saída 1 comuta para nível alto

A histerese previne oscilações espúrias e define os limites da excursão da rampa.

#### Bloco 4 — Chave de Reset Q1 (R5, R7, SW1)

O transistor **Q1 (BC547)** é acionado pela Saída 1 de U1:A através de **R7 (50 kΩ)**. Quando a Saída 1 vai para nível baixo, a base de Q1 perde polarização, Q1 corta e C1 carrega normalmente. Quando a Saída 1 vai para nível alto, Q1 satura e descarrega C1 rapidamente através de **R5 (10 kΩ)**, resetando a rampa quase instantaneamente.

**SW1** permite interromper esse caminho de reset manualmente para observar o comportamento do integrador isolado.

---

### 3.3 Etapas de carga e descarga

O ciclo de oscilação se divide em quatro etapas:

#### Etapa 1 — Carga de C1 (rampa sobe)

Vco (6 V) força corrente por R1 (100 kΩ) para C1. O integrador U1:B acumula carga e a Saída 2 sobe linearmente. Neste momento, Saída 1 está em nível alto e Q1 está **cortado** (SW1 fechada mas sem tensão de base suficiente).

$$\text{Slope} = \frac{V_{co}}{R_1 \cdot C_1} = \frac{6}{100 \times 10^3 \times 0{,}05 \times 10^{-6}} = 1200 \; \text{V/s}$$

#### Etapa 2 — Disparo em V_TH

A rampa atinge o limiar superior V_TH da entrada (+) de U1:A. O comparador comuta: **Saída 1 vai de alto para baixo**. Este evento aciona a base de Q1 via R7.

#### Etapa 3 — Reset de C1 (descarga rápida)

Q1 satura e descarrega C1 através de R5 (10 kΩ). A descarga é muito mais rápida que a carga — a rampa cai quase verticalmente de V_TH para V_TL. A Saída 1 permanece em nível baixo durante este instante.

#### Etapa 4 — Disparo em V_TL e reinício

Assim que a rampa cai até V_TL, U1:A comuta novamente: **Saída 1 vai para nível alto**, Q1 corta, e C1 começa a carregar de novo — iniciando um novo ciclo.

> 📷 **[INSERIR AQUI: Print do osciloscópio virtual do Proteus — Saídas 1 e 2 simultâneas]**

---

## 4. Limiares de Transição

### 4.1 Dedução genérica

O limiar de comutação de U1:A é determinado pela tensão na sua entrada (+), formada pelo divisor R3–R6 entre V2 e a própria Saída 1.

Pela superposição:

$$V_+ = V_2 \cdot \frac{R_6}{R_3 + R_6} + V_{out1} \cdot \frac{R_3}{R_3 + R_6}$$

O comparador comuta quando $V_{-} = V_+$, ou seja, quando a Saída 2 (rampa) atinge $V_+$.

**Limiar superior V_TH** — ocorre quando Saída 1 = V_sat_hi:

$$\boxed{V_{TH} = V_2 \cdot \frac{R_6}{R_3 + R_6} + V_{sat+} \cdot \frac{R_3}{R_3 + R_6}}$$

**Limiar inferior V_TL** — ocorre quando Saída 1 = V_sat_lo:

$$\boxed{V_{TL} = V_2 \cdot \frac{R_6}{R_3 + R_6} + V_{sat-} \cdot \frac{R_3}{R_3 + R_6}}$$

### 4.2 Cálculo numérico

O LM324 operando com alimentação de V1 = 10 V (single supply):
- $V_{sat+} \approx V_1 - 1{,}5 = 8{,}5 \; \text{V}$
- $V_{sat-} \approx 0 \; \text{V}$

$$V_{TH} = 2{,}5 \cdot \frac{100k}{51k + 100k} + 8{,}5 \cdot \frac{51k}{51k + 100k}$$

$$V_{TH} = 2{,}5 \times 0{,}6623 + 8{,}5 \times 0{,}3377 = 1{,}6556 + 2{,}8709 = \mathbf{4{,}53 \; V}$$

$$V_{TL} = 2{,}5 \cdot \frac{100k}{151k} + 0 \cdot \frac{51k}{151k} = \mathbf{1{,}66 \; V}$$

$$\Delta V = V_{TH} - V_{TL} = 4{,}53 - 1{,}66 = \mathbf{2{,}87 \; V}$$

### 4.3 Tabela de comparação — Limiares

| Grandeza     | Valor Teórico | Valor Simulado (Proteus) | Desvio (%) |
|--------------|:-------------:|:------------------------:|:----------:|
| V_TH         | 4,53 V        | _____                    | _____      |
| V_TL         | 1,66 V        | _____                    | _____      |
| ΔV (histerese)| 2,87 V       | _____                    | _____      |

> 📷 **[INSERIR AQUI: Print do cursor do osciloscópio medindo VTH e VTL no Proteus]**

---

## 5. Equação da Frequência

### 5.1 Dedução genérica

O período de oscilação é determinado pelo tempo que C1 leva para carregar de V_TL até V_TH com a corrente fornecida por Vco através de R1.

A corrente de carga é constante:

$$I = \frac{V_{co}}{R_1}$$

A tensão no capacitor cresce linearmente:

$$\Delta V = \frac{I \cdot T}{C_1} = \frac{V_{co} \cdot T}{R_1 \cdot C_1}$$

Isolando o período T (desprezando o tempo de reset, que é muito menor):

$$T = \frac{(V_{TH} - V_{TL}) \cdot R_1 \cdot C_1}{V_{co}}$$

Portanto, a frequência é:

$$\boxed{f = \frac{V_{co}}{(V_{TH} - V_{TL}) \cdot R_1 \cdot C_1}}$$

Esta equação mostra que **f é diretamente proporcional a Vco** — confirmando o comportamento de VCO (Voltage Controlled Oscillator).

### 5.2 Cálculo numérico (Vco = 6 V)

$$f = \frac{6}{2{,}87 \times 100 \times 10^3 \times 0{,}05 \times 10^{-6}}$$

$$f = \frac{6}{2{,}87 \times 5 \times 10^{-3}} = \frac{6}{0{,}01435} = \mathbf{418 \; Hz}$$

### 5.3 Tabela de comparação — Frequência

| Grandeza           | Valor Teórico | Valor Simulado (Proteus) | Desvio (%) |
|--------------------|:-------------:|:------------------------:|:----------:|
| Frequência (f)     | 418 Hz        | _____                    | _____      |
| Período (T)        | 2,39 ms       | _____                    | _____      |
| Tempo de subida    | ≈ 2,39 ms     | _____                    | _____      |
| Tempo de reset     | ≈ desprezível | _____                    | _____      |

---

## 6. Formas de Onda Teóricas

O gráfico abaixo mostra as formas de onda calculadas teoricamente para Vco = 6 V:

> 📷 **[INSERIR AQUI: formas_de_onda_teorico.png — gerado pelo script Python]**

**Parâmetros das formas de onda:**

| Saída   | Forma     | Amplitude          | Frequência | Observação               |
|---------|-----------|--------------------|------------|--------------------------|
| Saída 1 | Quadrada  | 0 V → 8,5 V        | 418 Hz     | Pulso estreito de reset  |
| Saída 2 | Rampa/tri | 1,66 V → 4,53 V    | 418 Hz     | Subida linear, queda abrupta |

---

## 7. Influência de Vco na Frequência

Substituindo na equação de frequência:

$$f(V_{co}) = \frac{V_{co}}{2{,}87 \times 10^{-2}} \quad [\text{Hz}]$$

| Vco (V) | f teórica (Hz) |
|:-------:|:--------------:|
| 1       | 70             |
| 2       | 140            |
| 3       | 209            |
| 4       | 279            |
| 6       | **418**        |
| 8       | 557            |
| 10      | 696            |

**Conclusão:** Vco controla linearmente a frequência de oscilação. O circuito funciona como um **VCO linear** (Voltage Controlled Oscillator).

> 📷 **[INSERIR AQUI: Print do Proteus com Vco variando (senóide ou triangular) — Questão 2]**

---

## 8. Questão 2 — Vco com Sinal Variável

Quando se aplica um sinal variável (senoidal ou triangular) em Vco, a frequência de oscilação acompanha instantaneamente o valor de Vco, produzindo um sinal **modulado em frequência (FM)**.

- **Vco senoidal →** a frequência da Saída 1 varia de forma senoidal em torno de f₀
- **Vco triangular →** a frequência da Saída 1 varia linearmente (chirp linear)

Este é o princípio de funcionamento de um **VCO analógico**, amplamente utilizado em PLLs (Phase-Locked Loops) e síntese de frequência.

> 📷 **[INSERIR AQUI: Print do Proteus — Vco senoidal + Saída 1 mostrando variação de frequência]**

---

## 9. Resumo dos Resultados

| Parâmetro        | Teórico    | Simulado | Desvio |
|------------------|:----------:|:--------:|:------:|
| V_TH             | 4,53 V     |          |        |
| V_TL             | 1,66 V     |          |        |
| ΔV (histerese)   | 2,87 V     |          |        |
| Frequência       | 418 Hz     |          |        |
| Período          | 2,39 ms    |          |        |

---

## 10. Referências

- Datasheet LM324 — Texas Instruments  
- Datasheet BC547 — Fairchild Semiconductor  
- SEDRA, A. S.; SMITH, K. C. *Microeletrônica*. 7ª ed. Pearson, 2015.  
- Simulação realizada no Proteus 8 Professional

---

*Documento gerado com auxílio de ferramentas computacionais. Equações em LaTeX, gráficos em Python/Matplotlib.*
