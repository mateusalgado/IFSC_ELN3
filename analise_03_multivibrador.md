# Análise 03 — Multivibradores com AmpOp

**Curso:** Tecnologia em Eletrônica Industrial / Engenharia Eletrônica  
**Unidade:** Eletrônica III  
**Professor:** Luis Carlos Martinhago Schlichting

---

## Objetivo

Analisar, simular e montar circuitos multivibradores baseados em amplificadores operacionais, determinando os limiares de transição, equações de frequência e o comportamento das saídas em função da tensão de controle Vco.

---

## 1. Circuito Multivibrador LM324

### 1.1 Esquemático

![Esquemático — Proteus](imgs/simulacao/q1_esquematico.png)

### 1.2 Parâmetros do circuito

| Componente | Valor |
|------------|-------|
| R1 | 100 kΩ |
| R2 | 51 kΩ |
| R3 | 51 kΩ |
| R4 | 51 kΩ |
| R5 | 10 kΩ |
| R6 | 100 kΩ |
| R7 | 50 kΩ |
| C1 | 0,05 µF |
| Q1 | BC547 |
| SW1 | SW-SPST |
| V1 | 10 V |
| V2 | 2,5 V |
| Vco | 6 V |
| U1:A, U1:B | LM324 |


### 1.3 Análise de Funcionamento

#### 1.3.1 Diagrama de blocos

![Diagrama de blocos do multivibrador](imgs/geradas/diagrama_blocos_q1.png)

O circuito opera como **oscilador** em malha fechada: o integrador (U1:B) gera a rampa, o comparador (U1:A) detecta os limiares e gera a onda quadrada, e Q1 reseta o integrador a cada ciclo.

---

#### 1.3.2 Comparador U1:A (Schmitt Trigger)

![Análise do comparador com histerese U1:A](imgs/schmitt.png)

U1:A é um **Schmitt Trigger** — comparador com realimentação positiva via R3 e R6. Dois limiares distintos são criados: quando a rampa (Saída 2) sobe e cruza $V_{TH}$, a Saída 1 vai a nível baixo; quando desce e cruza $V_{TL}$, volta a nível alto.

Pela superposição na entrada (+) de U1:A:

$$V_+ = V_2 \cdot \frac{R_6}{R_3 + R_6} + V_{out1} \cdot \frac{R_3}{R_3 + R_6}$$


##### Equações genéricas

$$V_{TH} = V_2 \cdot \frac{R_6}{R_3 + R_6} + V_{sat+} \cdot \frac{R_3}{R_3 + R_6}$$

$$V_{TL} = V_2 \cdot \frac{R_6}{R_3 + R_6} + V_{sat-} \cdot \frac{R_3}{R_3 + R_6}$$

##### Cálculo numérico

LM324 com alimentação única V1 = 10 V: $V_{sat+} \approx 8{,}95$ V e $V_{sat-} \approx 0$ V

$$V_{TH} = 2{,}5 \cdot \frac{100k}{151k} + 8{,}95 \cdot \frac{51k}{151k} = 1{,}66 + 3{,}02 = 4{,}68 \text{ V}$$

$$V_{TL} = 2{,}5 \cdot \frac{100k}{151k} + 0 = 1{,}66 \text{ V}$$

$$\Delta V = V_{TH} - V_{TL} = 3{,}02 \text{ V}$$

##### 1.3.2.1 Disparo em V_TH

Quando a Saída 2 atinge $V_{TH}$, U1:A comuta: Saída 1 vai a nível alto ($V_{sat+}$ ≈ 8,95 V). Essa tensão chega à base de Q1 via R5 (10 kΩ) e o satura.


##### 1.3.2.2 Disparo em V_TL

Quando a Saída 2 atinge $V_{TL}$, U1:A comuta de volta: Saída 1 vai a nível baixo, Q1 corta, e o ciclo recomeça.

---

#### 1.3.3 Integrador U1:B

![Análise do integrador U1:B](imgs/integrador.png)

U1:B está configurado como **integrador inversor** — C1 no caminho de realimentação. Com resistor no feedback o AmpOp seria um amplificador simples; com capacitor, a saída é a integral da entrada no tempo.

R4 (51 kΩ) vem de Vco e R2 (51 kΩ) vai para GND, formando um divisor **simétrico** na entrada (+) de U1:B. V1 (10 V) alimenta apenas o pino 4 do LM324, não entra no divisor.

$$V^+ = V_{co} \cdot \frac{R_2}{R_2 + R_4} = \frac{V_{co}}{2} = \frac{6}{2} = 3{,}0 \text{ V}$$

Pelo curto-circuito virtual, $V^- = V^+ = 3{,}0$ V. A corrente em R1 é:

$$I = \frac{V_{co} - V^+}{R_1} = \frac{6 - 3}{100k} = 30 \ \mu\text{A}$$

Essa corrente constante em C1 produz rampa linear com inclinação:

$$\frac{dV_{out}}{dt} = \frac{I}{C_1} = \frac{30 \ \mu}{0{,}05 \ \mu} = 600 \text{ V/s} = 0{,}6 \text{ V/ms}$$

---


#### 1.3.4 Chave Q1 e análise por estados

A chave Q1 não descarrega C1 diretamente — ela **desgarrega corrente no nó V⁻ de U1:B**, invertendo o sentido da integração. Para entender isso, analisamos o circuito em dois estados.

##### 1.3.4.1 — Carga (Q1 cortado)

Com Q1 cortado, o nó V⁻ de U1:B está em curto virtual com V⁺ = 3 V. A única corrente que chega ao nó vem de R1:

$$I_{carga} = \frac{V_{co} - V^+}{R_1} = \frac{6 - 3}{100k} = 30 \ \mu\text{A}$$

Essa corrente carrega C1, fazendo a **Saída 2 subir**:

$$\frac{dV_{out}}{dt} = +\frac{I_{carga}}{C_1} = +600 \text{ V/s}$$

Saída 1 permanece em nível baixo (Vsat_lo = 0 V) — a base de Q1 não recebe tensão suficiente via R5.

##### 1.3.4.2 Descarga (Q1 saturado)

Com Q1 saturado, o coletor vai a ≈ 0 V, e R7 (50 kΩ) passa a **drenar** corrente do nó V⁻:

$$I_{R7} = \frac{V^+}{R_7} = \frac{3}{50k} = 60 \ \mu\text{A}$$

R1 continua fornecendo 30 µA, mas R7 drena 60 µA — o déficit de 30 µA é suprido pelo próprio capacitor, fazendo a **Saída 2 descer**:

$$\frac{dV_{out}}{dt} = -\frac{I_{R7} - I_{carga}}{C_1} = -\frac{60 - 30}{C_1} \cdot 10^{-6} = -600 \text{ V/s}$$

A taxa de descarga é **idêntica em módulo** à taxa de carga — o que explica por que a Saída 2 é uma **onda triangular simétrica**.

---

### 1.4 Equação da Frequência

#### 1.4.1 Equações genéricas

O período é a soma do tempo de carga (Q1 cortado) e do tempo de descarga (Q1 saturado):

$$T = T_{carga} + T_{descarga}$$

A corrente de carga, com $V^+ = V_{co} \cdot \dfrac{R_2}{R_2+R_4} = V_{co}/2$ (pois R2 = R4):

$$I_{carga} = \frac{V_{co} - V^+}{R_1} = \frac{V_{co}}{2 R_1}$$

A corrente de descarga, quando Q1 satura e R7 drena o nó V⁻:

$$I_{descarga} = \left|\frac{V^+}{R_7} - I_{carga}\right| = \left|\frac{V_{co}}{2 R_7} - \frac{V_{co}}{2 R_1}\right|$$

$$T_{carga} = \frac{\Delta V \cdot C_1}{I_{carga}} \qquad T_{descarga} = \frac{\Delta V \cdot C_1}{I_{descarga}}$$

$$f = \frac{1}{T_{carga} + T_{descarga}}$$

#### Cálculo numérico (Vco = 1 V)

$$V^+ = \frac{V_{co}}{2} = 0{,}5 \text{ V}$$

$$I_{carga} = \frac{1 - 0{,}5}{100k} = 5 \ \mu\text{A}$$

$$I_{descarga} = \left|\frac{0{,}5}{50k} - 5\mu\right| = |10\mu - 5\mu| = 5 \ \mu\text{A}$$

Como $I_{carga} = I_{descarga} = 5\ \mu A$, a onda teórica é **triangular simétrica**:

$$T_{carga} = T_{descarga} = \frac{3{,}02 \times 0{,}05\mu}{5\mu} = 30{,}2 \text{ ms}$$

$$T = 30{,}2 + 30{,}2 = 60{,}4 \text{ ms} \quad \Rightarrow \quad f = 16{,}6 \text{ Hz}$$

#### Cálculo numérico (Vco = 6 V)

$$V^+ = \frac{V_{co}}{2} = 3{,}0 \text{ V}$$

$$I_{carga} = \frac{6 - 3}{100k} = 30 \ \mu\text{A}$$

$$I_{descarga} = \left|\frac{3}{50k} - 30\mu\right| = |60\mu - 30\mu| = 30 \ \mu\text{A}$$

Como $I_{carga} = I_{descarga} = 30\ \mu A$, a onda é **triangular simétrica**:

$$T_{carga} = T_{descarga} = \frac{3{,}02 \times 0{,}05\mu}{30\mu} = 5{,}03 \text{ ms}$$

$$T = 5{,}03 + 5{,}03 = 10{,}06 \text{ ms} \quad \Rightarrow \quad f = 99{,}4 \text{ Hz}$$

![Osciloscópio — Saída 1 e Saída 2, Vco = 8 V](imgs/simulacao/q1_vco_6.png)

A montagem em protoboard foi medida com o osciloscópio, mostrando limiares próximos aos valores teórico/simulado. A comparação foi feita com Vco constante em 6 V.

![Montagem em protoboard — limiares e rampa](imgs/protoboard/vco6v.png)

![Montagem em protoboard — tempo e frequência](imgs/protoboard/vco6t.png)


> O protótipo confirma a operação do Schmitt Trigger e do integrador em Vco constante: a rampa triangular e os dois limiares aparecem tanto na simulação quanto na montagem, com pequeno desvio devido a condições reais de bancada.

####  Comparação — Frequência, Período e Limiar

| Grandeza | Teórico | Simulado | Montagem | Desvio T×S |
|----------|:-------:|:--------:|:--------:|:----------:|
| T_carga | 5,03 ms | ≈ 4,98 ms | ≈ 5,5 ms | −0,8% |
| T_descarga | 5,03 ms | ≈ 4,98 ms | ≈ 5,5 ms | −0,8% |
| T total | 10,06 ms | 9,96 ms | ≈ 10,10 ms | −0,0% |
| f | 99,4 Hz | 100,4 Hz | ≈ 17,0 Hz | +1,0% |
| V_TH | 4,68 V | 4,68 V | ≈ 4,64 V | 0,0% |
| V_TL | 1,66 V | 1,65 V | ≈ 1,52 V | −0,3% |
| ΔV | 3,02 V | 3,03 V | ≈ 3,12 V | +0,3% |



> **Sobre o desvio de frequência:** o modelo teórico de carga/descarga simétrica concorda excelentemente com o simulado (desvio de apenas ±0,8%). O pequeno desvio residual deve-se ao modelo SPICE do LM324 — corrente de bias (~45 nA), tensão de offset (~±7 mV) e tempo de resposta finito do comparador. Os limiares V_TH e V_TL também concordam excelentemente (< 1%), confirmando a validade das equações do Schmitt Trigger.

---

### 1.5 Influência de Vco na Frequência

A expressão de frequência deve ser feita com os nomes dos componentes que determinam cada termo: o resistor de carga $R_1$, o resistor de descarga $R_7$, o capacitor de integração $C_1$ e a faixa de comutação do Schmitt Trigger $\Delta V = V_{TH} - V_{TL}$.

A tensão no nó positivo do integrador é fixada pelo divisor resistivo formado por $R_2$ e $R_4$:

$$V^+ = V_{co} \cdot \frac{R_2}{R_2 + R_4}$$

No nosso circuito, $R_2 = R_4$, logo $V^+ = V_{co}/2$.

#### 1.5.1 Dedução da Fórmula de Frequência

**Passo 1:** Corrente de carga do capacitor $C_1$.

Quando Q1 está cortado, o nó $V^-$ do integrador está em curto virtual com $V^+$, e a corrente que sai de $V_{co}$ por $R_1$ integra no capacitor:

$$I_{carga} = \frac{V_{co} - V^+}{R_1} = \frac{V_{co} - V_{co}/2}{R_1} = \frac{V_{co}}{2R_1}$$

**Passo 2:** Corrente de descarga do capacitor $C_1$.

Quando Q1 está saturado, a corrente de descarga no nó $V^-$ é dada pela diferença entre a corrente em $R_7$ e a corrente de carga que ainda entra por $R_1$:

$$I_{descarga} = \left|\frac{V^+}{R_7} - I_{carga}\right| = \left|\frac{V_{co}/2}{R_7} - \frac{V_{co}}{2R_1}\right|$$

Esta expressão deixa claro quais componentes influenciam a descarga: $R_7$ controla a retirada de corrente do nó do integrador, enquanto $R_1$ continua fornecendo corrente de carga.

**Passo 3:** Caso específico do circuito.

Com $R_1 = 100\ \text{k}\Omega$ e $R_7 = 50\ \text{k}\Omega$, temos:

$$I_{descarga} = \frac{V_{co}}{2} \left| \frac{1}{R_7} - \frac{1}{R_1} \right| = \frac{V_{co}}{2} \left| \frac{1}{50\text{k}} - \frac{1}{100\text{k}} \right| = \frac{V_{co}}{2R_1}$$

Portanto, neste projeto específico, as correntes de carga e descarga coincidem numericamente, mas isso é consequência direta da escolha de $R_1$ e $R_7$.

**Passo 4:** Tempo de variação do capacitor.

A variação de tensão $\Delta V$ sobre $C_1$ entre os limiares do Schmitt Trigger define os tempos de carga e descarga:

$$T_{carga} = \frac{\Delta V \cdot C_1}{I_{carga}} = \frac{\Delta V \cdot C_1}{V_{co}/(2R_1)} = \frac{2 R_1 \cdot \Delta V \cdot C_1}{V_{co}}$$

$$T_{descarga} = \frac{\Delta V \cdot C_1}{I_{descarga}} = \frac{\Delta V \cdot C_1}{\left|\dfrac{V_{co}/2}{R_7} - \dfrac{V_{co}}{2R_1}\right|}$$

Estas expressões mostram claramente a dependência de $T_{carga}$ em $R_1$, $C_1$ e $\Delta V$, e a dependência de $T_{descarga}$ em $R_7$, $R_1$, $C_1$ e $\Delta V$.

**Passo 5:** Período e frequência do oscilador.

O período total é a soma dos tempos de carga e descarga:

$$T = T_{carga} + T_{descarga} = \frac{2 R_1 \cdot \Delta V \cdot C_1}{V_{co}} + \frac{\Delta V \cdot C_1}{\left|\dfrac{V_{co}/2}{R_7} - \dfrac{V_{co}}{2R_1}\right|}$$

$$f = \frac{1}{T} = \frac{1}{\dfrac{2 R_1 \cdot \Delta V \cdot C_1}{V_{co}} + \dfrac{\Delta V \cdot C_1}{\left|\dfrac{V_{co}/2}{R_7} - \dfrac{V_{co}}{2R_1} \right|}}$$

Esta é a fórmula geral de frequência em função dos componentes do circuito: $R_1$, $R_7$, $C_1$ e $\Delta V$, além de $V_{co}$ e da divisão $V^+ = V_{co} R_2/(R_2+R_4)$.

**Passo 6:** Fórmula final para este projeto.

Com $R_2 = R_4$ e $R_7 = R_1/2$, o termo de descarga simplifica e o período total fica:

$$T = \frac{4 R_1 \cdot \Delta V \cdot C_1}{V_{co}}$$

$$\boxed{f(V_{co}) = \frac{V_{co}}{4\,R_1\,\Delta V\,C_1}}$$

Nesta forma final, cada elemento está identificado:

- $R_1$: resistor de entrada que define $I_{carga}$
- $R_7$: resistor de descarga que define $I_{descarga}$
- $C_1$: capacitor de integração que determina a constante de tempo
- $\Delta V$: faixa de trabalho do Schmitt Trigger, definida por $R_3$, $R_6$ e $V_2$

**Conclusão:** A frequência do VCO é diretamente proporcional a $V_{co}$ e inversamente proporcional a $R_1$, $C_1$ e $\Delta V$. A igualdade numérica entre $I_{carga}$ e $I_{descarga}$ só aparece porque o projeto usa $R_7 = R_1/2$; se o valor de $R_7$ fosse diferente, a expressão geral de $T_{descarga}$ e, portanto, de $f$, teria dependência explícita em $R_7$.

Na prática, a operação confiável do multivibrador exige que a corrente de integração não fique da mesma ordem das correntes de polarização e fuga do LM324 e do BC547. Adotando uma margem mínima de cerca de $10\ \mu\text{A}$, obtemos um Vco prático mínimo:

$$V_{co,min} \approx 2 R_1 \cdot 10\ \mu\text{A} = 2{,}0 \text{ V}$$

Abaixo desse valor, o circuito pode até ter uma solução teórica, mas passa a ser instável na prática, porque o nó de referência do integrador fica muito baixo e a rampa se torna muito lenta para garantir comutação confiável nos limiares $V_{TL} \approx 1{,}66$ V e $V_{TH} \approx 4{,}68$ V.

| Vco (V) | I_carga = I_desc (µA) | T_carga = T_desc (ms) | f (Hz) |
|:-------:|:----------------------:|:----------------------:|:------:|
| 2 | 10 | 15,1 | 33,1 |
| 3 | 15 | 10,1 | 50,1 |
| 4 | 20 | 7,55 | 66,2 |
| 6 | 30 | 5,03 | **99,4** |
| 8 | 40 | 3,78 | 132,5 |
| 10 | 50 | 3,02 | 165,6 |

O circuito funciona como **VCO (Voltage Controlled Oscillator)** com resposta **linear**: f é diretamente proporcional a Vco, e a forma de onda da Saída 2 permanece triangular simétrica em toda a faixa.


---

### 1.6 Vco com Sinal Variável

Quando se aplica um sinal variável em Vco, a corrente de integração $I = V_{co}/(2R_1)$ varia instantaneamente, fazendo a frequência de oscilação acompanhar o sinal:

- **Vco senoidal** → frequência da Saída 1 varia senoidalmente em torno de f₀ — modulação em frequência (FM)
- **Vco triangular** → frequência varia linearmente no tempo — chirp linear

Este é o princípio de funcionamento de VCOs analógicos em PLLs (Phase-Locked Loops).

![Simulação — Vco senoidal, vista de amplitude e tempo](imgs/simulacao/q1_vco_sen.png)

![Montagem em protoboard — Vco senoidal, vista de amplitude](imgs/protoboard/vcosenv.png)

![Montagem em protoboard — Vco senoidal, vista de tempo](imgs/protoboard/vcosent.png)

![Montagem em protoboard — Vco triangular, vista de amplitude](imgs/protoboard/vcotriv.png)

![Montagem em protoboard — Vco triangular, vista de tempo](imgs/protoboard/vcotrit.png)


---

## Conclusão

O circuito é estruturado em **malha fechada** com três blocos:

- **Comparador U1:A (Schmitt Trigger):** Define dois limiares de comutação bem definidos ($V_{TH} = 4{,}68$ V e $V_{TL} = 1{,}66$ V) com histerese de $\Delta V = 3{,}02$ V.

- **Integrador U1:B:** Transforma a corrente constante do divisor Vco em rampa linear. O divisor simétrico (R2 = R4) garante que $V^+ = V_{co}/2$, e o capacitor C1 no caminho de realimentação produz taxa de integração $\frac{dV}{dt} = I / C_1$.

- **Chave Q1:** Não descarrega C1 diretamente — ela **inverte o sentido da corrente de integração**.

A descoberta mais importante é que **$I_{carga} = I_{descarga}$ para qualquer valor de Vco**. Isto ocorre porque:

- R1 e R7 foram escolhidos com a relação R1 = 2 × R7 (100 kΩ vs 50 kΩ)
- Ambas as correntes escalam proporcionalmente com Vco

Resultado: a onda permanece **triangular simétrica** independentemente da tensão de controle.

A proporção é perfeita: dobrar Vco dobra a frequência. Este é o comportamento ideal de um **VCO (Voltage Controlled Oscillator)** e é a base teórica para aplicações em **Phase-Locked Loops (PLLs)** e **sínteses de frequência**.

#### 4. Validação Experimental: Concordância Teórico/Simulado/Prático

| Parâmetro | Desvio Teórico-Simulado | Qualidade |
|-----------|:-----:|:----------:|
| Período | ±0,8% | **Excelente** |
| Frequência | ±1,0% | **Excelente** |
| $V_{TH}$ | 0% | **Perfeita** |
| $V_{TL}$ | −0,3% | **Excelente** |
| $\Delta V$ | +0,3% | **Excelente** |

Os pequenos desvios residuais (< 1%) são atribuídos ao modelo SPICE do LM324.

### Aplicações e Relevância

Este tipo de VCO é fundamental em:

1. **Síntese de frequência:** Gerar tons ou sinais em faixa controlada
2. **Phase-Locked Loops (PLLs):** Sincronização de fase em comunicações e processamento de sinais
3. **Modulação em frequência (FM):** Já demonstrada com Vco senoidal e triangular
4. **Gerador de rampa controlável:** Para aplicações de amostragem e varredura

---


## Referências

- Datasheet LM324 — Texas Instruments
- Datasheet TL082 — Texas Instruments
- Datasheet BC547 / BC548 — Fairchild Semiconductor
- SEDRA, A. S.; SMITH, K. C. *Microeletrônica*. 7ª ed. Pearson, 2015.
- Simulação: Proteus 8 Professional

---

*Equações em LaTeX, gráficos em Python/Matplotlib.*
