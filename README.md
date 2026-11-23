# Minicurso Godot SASI — Base dos Alunos

Repositório inicial do minicurso **Desvendando a Godot: Crie seu primeiro jogo** (SASI 2026 · LASI / UFAC). Ele reúne os recursos visuais e sonoros do pacote **Ninja Adventure Asset Pack** (Pixel-boy, licença CC0) e a configuração mínima do projeto na Godot para construirmos o jogo do zero durante a aula.

## Como baixar e abrir na Godot 4.7.2

1. Baixe este repositório pelo botão **Code > Download ZIP** (ou clone com `git clone`).
2. **Extraia a pasta** do `.zip` antes de abrir o projeto (não abra diretamente de dentro do arquivo compactado).
3. Abra a **Godot 4.7.2**, clique em **Importar** no Gerenciador de Projetos, selecione o arquivo `project.godot` dentro da pasta extraída e confirme.
4. Opcional (interface em português): vá em **Editor → Configurações do Editor... → Geral → Interface → Editor → Localização → Idioma do Editor**, selecione `[pt_BR] Portuguese, Brazil` e clique em **Salvar e Reiniciar**.
5. Em **Projeto → Configurações do Projeto... → Mapa de Entrada**, crie as ações que vamos usar nos scripts:
   - `move_left` (`A`, `Seta esquerda`)
   - `move_right` (`D`, `Seta direita`)
   - `move_up` (`W`, `Seta cima`)
   - `move_down` (`S`, `Seta baixo`)
   - `attack` (`Espaço`)

   A ação nativa `ui_cancel` já vem configurada com `Esc`; não é preciso criá-la.

## Etapas da aula

### 1. TileSet e cenário da vila
1. Crie `cenario/terreno.tscn` com raiz `Terreno` (`Node2D`, `y_sort_enabled` ativado). Adicione `Chao` (`TileMapLayer`, `z_index = -3`).
2. No Inspetor de `Chao`, em **Tile Set**, crie **Novo TileSet** (grade `16 × 16`) e use **Salvar como...** para gravar `cenario/tileset.tres`. No TileSet, em **Camadas de Física → Adicionar Elemento**, configure a `Camada de Física 0` com `collision_layer = 1` e `collision_mask = 2`.
3. Na aba inferior **TileSet → Tiles**, arraste `solo.png`, `agua.png`, `natureza.png`, `casas.png`, `piso.png` e `parede.png` de `assets/cenario/` para o painel de fontes (*Atlas Source*). Confirme **Sim** para criar os tiles automaticamente.
4. Em **TileSet → Pintar**, selecione **Camada de Física 0**. Com o editor de polígono em foco, pressione `F` ou use o menu de redefinição para criar a forma padrão. Confira o quadrado `16 × 16` no painel central; depois clique nos tiles de arbusto, água e parede que devem ter colisão.
5. Adicione a `Terreno` as camadas `Agua` (`z_index = -2`), `ChaoFlorido` (`z_index = -1`, opcional), `Natureza` e `Casas` (opcional), todas `TileMapLayer` com o mesmo `tileset.tres`. Ative `y_sort_enabled` em `Natureza` e `Casas`.
6. Selecione cada camada filha para abrir **TileMap** e pintar. Em `Chao`, preencha `32 × 20` células de `(0, 0)` até `(31, 19)`. Em `Natureza`, feche toda a borda com arbustos com colisão. Deixe livre de obstáculos e água a área do spawn `(232, 178)`, células `(13..16, 10..12)`, e pinte água com colisão fora dela.
7. Crie `fases/nivel_vila.tscn` com raiz `Vila` (`Node2D`, `y_sort_enabled` ativado) e instancie `cenario/terreno.tscn` (`Ctrl+Shift+A`).

### 2. Jogador e movimento
1. Crie `entidades/jogador/jogador.tscn` com raiz `Jogador` (`CharacterBody2D`, `Motion Mode = Floating`, colisão `Layer = 2`, `Mask = 1`). Adicione `Sprite2D` em `(0, -6)` com `assets/personagens/hunter.png`, `Hframes = 4` e `Vframes = 7`; `CollisionShape2D` em `(0, -1)` com `CircleShape2D` de raio `5`; `AnimationPlayer` e `Camera2D`.
2. Selecione `AnimationPlayer` e abra **Animação**. Para cada clipe, use **Animação → Nova...**, ajuste duração e loop. Antes de gravar, configure o ajuste (snap) em segundos com passo `0,01 s` e confira os tempos na régua.
3. Em cada tempo da tabela, selecione `Sprite2D`, ajuste `Animation → Frame` e clique na chave ao lado de `Frame`. Na primeira chave, confirme a faixa `Sprite2D:frame` e mantenha atualização discreta. Crie agora os clipes `idle_*` e `walk_*`; os de ataque entram na etapa 3.

| Clipe | Duração | Loop | Tempos (s) | Quadros |
| :--- | :--- | :--- | :--- | :--- |
| `idle_down` | 0.24 s | Sim | 0.00 | 0 |
| `idle_up` | 0.24 s | Sim | 0.00 | 1 |
| `idle_left` | 0.24 s | Sim | 0.00 | 2 |
| `idle_right` | 0.24 s | Sim | 0.00 | 3 |
| `walk_down` | 0.52 s | Sim | 0.00, 0.13, 0.26, 0.39 | 4, 0, 12, 0 |
| `walk_up` | 0.52 s | Sim | 0.00, 0.13, 0.26, 0.39 | 5, 1, 13, 1 |
| `walk_left` | 0.52 s | Sim | 0.00, 0.13, 0.26, 0.39 | 6, 2, 14, 2 |
| `walk_right` | 0.52 s | Sim | 0.00, 0.13, 0.26, 0.39 | 7, 3, 15, 3 |
| `attack_down` | 0.24 s | Não | 0.00, 0.08, 0.16 | 0, 16, 16 |
| `attack_up` | 0.24 s | Não | 0.00, 0.08, 0.16 | 1, 17, 17 |
| `attack_left` | 0.24 s | Não | 0.00, 0.08, 0.16 | 2, 18, 18 |
| `attack_right` | 0.24 s | Não | 0.00, 0.08, 0.16 | 3, 19, 19 |

4. Com o instrutor, anexe `entidades/jogador/jogador.gd` a `Jogador` e implemente movimento e animação usando as quatro ações `move_*` e `move_and_slide()`.
5. Em `fases/nivel_vila.tscn`, instancie `jogador.tscn` em `(232, 178)` e anexe `fases/fase.gd` em `Vila` para limitar a `Camera2D` à área `512 × 320`.
6. Salve todas as cenas (`Ctrl+Shift+S`), volte à aba `fases/nivel_vila.tscn` e teste com `F6`.

### 3. Ataque com Espaço
1. No `AnimationPlayer`, crie os quatro clipes `attack_*` da tabela, sem loop. Grave também as chaves de quadros repetidos em `0.08` e `0.16 s`.
2. Em `jogador.gd`, leia `Input.is_action_just_pressed("attack")` com um tempo de recarga (`cooldown`), trave o deslocamento enquanto a animação toca e execute a animação da direção atual com `play(...)`.
3. Salve (`Ctrl+S`) e teste o ataque nas quatro direções em `nivel_vila.tscn` (`F6`).

### 4. Menu inicial
1. Crie `interfaces/menu_principal/menu_principal.tscn` com raiz `MenuPrincipal` (`Control`, âncoras **Retângulo Completo**). Adicione `Cartao` (`PanelContainer`) como filho de `MenuPrincipal` e `Coluna` (`VBoxContainer`) como filha de `Cartao`. Dentro de `Coluna`, adicione `Titulo` (`Label`, `Text = "Trilhas da Vila"`) e os botões `NovoJogo` (`Text = "Novo Jogo"`) e `Sair` (`Text = "Sair"`).
2. Crie e anexe `interfaces/menu_principal/menu_principal.gd` à raiz `MenuPrincipal`. Conecte o sinal `pressed` dos botões: `NovoJogo` abre `res://fases/nivel_vila.tscn` e `Sair` encerra o jogo.
3. Defina `res://interfaces/menu_principal/menu_principal.tscn` como **Cena Principal** em **Projeto → Configurações do Projeto...** e teste com `F5`.

### 5. Menu de pausa
1. Crie `interfaces/menu_pausa/menu_pausa.tscn` com raiz `MenuPausa` (`CanvasLayer`, `layer = 4`, `visible = false`, **Node → Process → Mode = Always**). Adicione `Fundo` (`ColorRect`) e, dentro dele, `Centro` (`CenterContainer`), ambos com âncoras **Retângulo Completo**. Em `Centro`, adicione `Cartao` (`PanelContainer`) → `Coluna` (`VBoxContainer`) → `Titulo` (`Label`, `Text = "Jogo pausado"`), `Continuar` (`Button`, `Text = "Continuar"`) e `MenuPrincipal` (`Button`, `Text = "Menu Principal"`).
2. Crie e anexe `interfaces/menu_pausa/menu_pausa.gd` à raiz `MenuPausa`. Faça `ui_cancel` (`Esc`) alternar juntos a pausa e a visibilidade do menu. Conecte os botões: `Continuar` despausa e esconde o menu; `MenuPrincipal` despausa e abre `res://interfaces/menu_principal/menu_principal.tscn`.
3. Instancie `menu_pausa.tscn` dentro de `fases/nivel_vila.tscn`, salve todas as cenas (`Ctrl+Shift+S`) e teste `F5`.

### 6. Exportar e publicar no itch.io
1. Confira se os **Modelos de Exportação 4.7.2** estão instalados. Em **Projeto → Exportar...**, clique em **Adicionar... → Windows Desktop**, deixe **Embed PCK** desativado e clique em **Exportar Projeto...**. Salve em `build/windows/minicurso.exe`, com o `.pck` na mesma pasta.
2. Compacte `.exe` e `.pck` em um `.zip`, extraia em outra pasta e teste o executável fora da Godot.
3. No **itch.io**, crie um projeto *Downloadable*, envie o `.zip`, marque a plataforma **Windows** e salve ou publique.

## Depois da aula

Ideias para continuar praticando em casa usando os extras que já estão em `assets/`:
- Criar um objeto interativo usando `assets/objetos/bau.png` e tocar `assets/sons/tesouro.wav` com `AudioStreamPlayer`.
- Adicionar um NPC na vila usando `assets/personagens/aldeao.png` e exibir uma fala simples na tela.
- Criar uma segunda fase e conectar as duas com uma `Area2D` de passagem.

Links úteis para continuar estudando:
- Documentação oficial da Godot em português: [docs.godotengine.org/pt-br/4.x/](https://docs.godotengine.org/pt-br/4.x/)
- Curso interativo de GDScript: [GDQuest Learn GDScript From Zero](https://gdquest.github.io/learn-gdscript/)
- Pacotes gratuitos de arte e áudio (CC0): [Kenney.nl](https://kenney.nl/)

## Créditos

Arte e efeito sonoro: **Ninja Adventure Asset Pack**, de Pixel-boy. Licença: Creative Commons Zero (CC0 1.0 Universal) — [pixel-boy.itch.io/ninja-adventure-asset-pack](https://pixel-boy.itch.io/ninja-adventure-asset-pack).
