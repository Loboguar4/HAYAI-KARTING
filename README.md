
# HAYAI KARTING: Traction Lab

<https://loboguar4.github.io/HAYAI-KARTING/>

**Alpha 0.5 — Modo Treino Livre**

> *Hayai* (速い) — japonês: rápido, veloz.

Um simulador de kart baseado em browser que funde a física real de um kart **IAME X30 125cc** com elementos originais de gameplay. Escrito inteiramente em HTML5 + JavaScript puro, sem dependências externas, sem servidor — abre diretamente no navegador.

---

## Objetivo

Fundir aspectos de pilotagem real de um kart IAME X30 (ou similar) com elementos originais — **sistema de agressividade**, **stamina** e **slipstream exagerado** — dentro de uma webengine leve baseada em **Mode7**, escrita em JavaScript e HTML5 vanilla.

Os fundamentos desta versão são **tracionamento** (understeer, oversteer e counter-steer) e **linha de corrida** (traçado e apex).

---

## Filosofia

**Máxima acessibilidade.** O jogo deve rodar em (quase) qualquer navegador web moderno, sem instalação, sem plugins, sem login. Um único arquivo `.html` é suficiente.

---

## Como jogar

Abra `index.html` diretamente no navegador. Nenhuma dependência externa é necessária.

### Controles — Teclado

| Tecla | Ação |
|---|---|
| `↑` | Acelerar |
| `↓` | Freiar |
| `←` / `→` | Esterço |
| `A` + `←` / `→` | Esterço agressivo (excede limite normal) |
| `A` + `↓` | Freio agressivo |
| `A` + `↑` | Slipstream (vácuo simulado) |
| `L` | Mostrar/ocultar painel de voltas |
| `F` | Alternar fullscreen |

### Controles — Mouse (opcional)

| Input | Ação |
|---|---|
| Botão direito | Acelerar |
| Botão esquerdo | Freiar |
| Movimento X | Esterçamento analógico |
| `A` | Agressividade |

Ative o controle por mouse no menu e ajuste a sensibilidade pelo slider.

---

## Sistemas físicos implementados

### Motor — IAME Parilla X30 125cc
Curva de torque modelada por gaussiana calibrada (~28 Nm de pico a 11.000 rpm), com shoulder secundário em ~8.500 rpm representando o comportamento real do 2T. O motor reproduz comportamento de ralenti a 3.200 rpm, power band entre 9.500–13.500 rpm e temperatura de freio afetando a eficácia.

### Modelo de pneu — Bicycle Model + Hybrid Tyre
Baseado no modelo de bicicleta clássico (dois eixos, slip angle frontal e traseiro). O modelo de pneu é híbrido: linear até 0,28 rad (16°) de slip angle, com queda exponencial além do pico (`k = 0,55/rad`). Parâmetros: `Cf = 12.000 N/rad`, `Cr = 11.000 N/rad`.

### Transferência de peso dinâmica
CG baixo do kart (`h/wb = 0,22/1,05 = 0,210`). Frenagem pesada reduz o grip traseiro em até 17%; aceleração carrega levemente o traseiro.

### Tração — Understeer / Oversteer / Counter-steer
- **Understeer:** saturação dianteira reduz o Mz efetivo (underFactor).
- **Oversteer:** limiar baseado em slip angle traseiro; buildRate usa curva de potência `excess^1,4` para criar zona macia antes do spin abrupto.
- **Counter-steer:** momento restaurador físico (`Mz_cs`) proporcional a `csEffect × slideR × velocidade`.
- **Fricção viscosa de guinada** (`kFricYaw = 1,8`): limita spins a 150–270° sem cap artificial.

### Lock-up de freio
Acumulador de stress determinístico: `brakeStress` sobe durante frenagem pesada em alta velocidade, trava quando atinge 1,0. Pumping (soltar o freio) drena 4× mais rápido.

### Temperatura (em fase de aprimoramento)
- **Pneus:** aquecimento por forças laterais, frenagem e aceleração. Grip ótimo entre 72–92°C; acima de 118°C o grip degrada.
- **Freios:** aquecimento proporcional à velocidade. Eficácia cresce até 280°C, fade começa após 600°C.

### Stamina
Tecla `[A]` consome resistência em ~3s. Recuperação passiva em ~25s. Abaixo de 25% entra na "zona vermelha" — 5%: `[A]` bloqueado e mensagem "CANSADO!".

### Slipstream
`[A]` + `↑` sem curva acumula vácuo (0→1 em ~0,5s), com bônus de velocidade e visual de partículas azuis.

### Sistema de áudio — Web Audio API
Síntese procedural de motor 2T por **pulsos de combustão discretos** (1 pulso/revolução). Camadas: crank (dente-de-serra grave), vento aerodinâmico, rolamento de pneus, chiado de derrapagem, chocalho de zebra, mecânico (corrente). Sons one-shot para batidas em barreiras e solavancos de zebra.

### Pista e renderização
- **Mode7** com textura de mundo 9600×9600px e perspectiva por scanline.
- Catmull-Rom spline com 4.200 pontos, 32 pontos de controle.
- Barreiras de pneus com colisão OBB por coluna.
- Zebras com tipos e posições geradas automaticamente por curvatura.
- 2 checkpoints de setor (S1 a 45%, S2 a 85%) + linha de chegada.
- Minimapa gerado em canvas offscreen.

---

## Configurações in-game

Acessíveis pelo menu antes de largar:

| Slider | Range | Padrão | Efeito |
|---|---|---|---|
| Controle de Tração (TC) | 0–10 | Desligado | Corte de engine proporcional ao slideR e cornerThrottle |
| Sensibilidade de Esterço | 1–10 | 5 (médio) | Multiplica PHY.steerSens de 0,60× a 1,50× |

---

## Constantes físicas de referência

```
Massa total (piloto + kart):  165 kg
Distância entre eixos:         1,05 m
CG bias (frente):             42%
Momento de inércia yaw (Iz):  18,0 kg·m²
Potência de pico:             ~33 cv @ 13.000 rpm
Torque de pico:               28 Nm @ 11.000 rpm
Velocidade máxima:            140 km/h (38,89 m/s)
Desaceleração máxima:         32 m/s²
```

---

## Versão publicada

**Alpha 0.5 — Treino Livre.** Apenas modo de treino livre com uma pista disponível, sem adversários. Voltas válidas exigem passagem pelos dois checkpoints de setor.

---

## Roadmap

Funcionalidades planejadas para versões futuras, sem ordem ou prazo definidos:

- [ ] Calibragem e aperfeiçoamento contínuo dos sistemas físicos e de tração
- [ ] Marcação de frenagem estática (padrão de pista) e dinâmica (melhor ponto da sessão) no asfalto
- [ ] Renderização do kart inteiro com câmera em terceira pessoa
- [ ] Voltas fantasmas (ghost lap)
- [ ] IA adversária
- [ ] Mais pistas e karts
- [ ] Modo Corrida
- [ ] Modo Online

---

## Licença

Distribuído sob a licença **Apache 2.0**. Veja `LICENSE` para detalhes.

---

## Autoria

**Bandeirinha**

---

Para apoiar esse e mais projetos independentes:

<https://www.github.com/Loboguar4>

<https://www.youtube.com/@esc-SubDev>

<https://www.pixgg.com/bandeirinha>

---

*HAYAI KARTING: Traction Lab — Alpha 0.5*
