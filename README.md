# TWSDK

Biblioteca JavaScript utilitária para o jogo Tribal Wars PRO (TWPRO).
Cada módulo é carregado separadamente e expõe APIs globais sob `window.TWSDK_*`.
Ao final, [`twSDK-Core.js`](twSDK/twSDK-Core.js:1) agrega todos os módulos em um objeto único `window.TWSDK`.

> Objetivo

Fornecer uma coleção de utilitários, helpers e caches para facilitar a escrita de scripts, userscripts e ferramentas auxiliares que interajam com o jogo, evitando duplicação de lógica (ex.: parsing de coordenadas, cálculos de distância, cache massivo de world data) e padronizando acesso a dados comuns.

## Conteúdo do SDK (visão geral dos módulos)

- [`twSDK-Helpers.js`](twSDK/twSDK-Helpers.js:1)
  - Funções utilitárias de baixo nível: acesso seguro a `game_data`, parsing e validação, wrappers de logging e helpers para manipulação DOM específica do jogo.
- [`twSDK-Constants.js`](twSDK/twSDK-Constants.js:1)
  - Constantes estáticas do jogo: pontos por nível de edifício, produção base de recursos, regex para coordenadas, IDs fixos e outros valores que raramente mudam.
- [`twSDK-Utils.js`](twSDK/twSDK-Utils.js:1)
  - Utilitários comuns: cálculo de distância entre coordenadas, conversão entre formatos, formatação de tempo e helpers para conversões geográficas.
- [`twSDK-Player.js`](twSDK/twSDK-Player.js:1)
  - Acesso e transformação de dados do jogador atual (nome, pontos, status, recursos, features habilitadas).
- [`twSDK-Village.js`](twSDK/twSDK-Village.js:1)
  - Helpers para a aldeia atual, seleção multi-aldeia e integração com gerenciadores locais (ex.: TWPROVillageManager).
- [`twSDK-Combat.js`](twSDK/twSDK-Combat.js:1)
  - Modelos e cálculos de combate: estatísticas de unidades, composição de ataque/defesa, estimativas de perdas e simuladores simples.
- [`twSDK-Research.js`](twSDK/twSDK-Research.js:1)
  - Dados e helpers relacionados a pesquisas e ao status do herói/cavaleiro/paladino.
- [`twSDK-Buildings.js`](twSDK/twSDK-Buildings.js:1)
  - Acesso à fila de construções, cálculo de prioridades e recomendações para próximos edifícios.
- [`twSDK-Economy.js`](twSDK/twSDK-Economy.js:1)
  - Inventário, gerenciamento de bônus/efeitos, farm helper e lógica de quests/diárias.
- [`twSDK-Awards.js`](twSDK/twSDK-Awards.js:1)
  - Captura e processamento de desafios/awards do DOM ou via fetch ao backend do jogo.
- [`twSDK-WorldData.js`](twSDK/twSDK-WorldData.js:1)
  - Download, normalização e cache em IndexedDB de grandes conjuntos (aldeias, jogadores, tribos). APIs baseadas em Promises para consultas e filtros.
- [`twSDK-UI.js`](twSDK/twSDK-UI.js:1)
  - Componentes e helpers visuais reutilizáveis (modais, notificações, painéis e helpers para injetar UI no jogo).
- [`twSDK-Core.js`](twSDK/twSDK-Core.js:1)
  - Agregador: monta `window.TWSDK` e fornece utilitários de inicialização, verificação de readiness, debug e entry points comuns.

## Como carregar / ordem recomendada

1. Carregar os módulos na ordem indicada nas importações de [`twSDK-Core.js`](twSDK/twSDK-Core.js:1). Em userscripts (Tampermonkey/Violentmonkey) use `@require` com a mesma ordem: Helpers -> Constants -> Utils -> ... -> Core (Core por último).
2. Garantir que o contexto do jogo (`game_data`) esteja disponível antes de usar APIs que dependam dele. Use `window.TWSDK.isReady()` quando disponível.

Exemplo de cabeçalho Tampermonkey (pseudocódigo):

```js
// ==UserScript==
// @name         Meu Script com TWSDK
// @namespace    http://tampermonkey.net/
// @version      0.1
// @match        https://*.tribalwars.*/*
// @require      https://meu.cdn/twSDK/twSDK-Helpers.js
// @require      https://meu.cdn/twSDK/twSDK-Constants.js
// @require      https://meu.cdn/twSDK/twSDK-Utils.js
// @require      https://meu.cdn/twSDK/twSDK-Player.js
// @require      https://meu.cdn/twSDK/twSDK-Village.js
// @require      https://meu.cdn/twSDK/twSDK-WorldData.js
// @require      https://meu.cdn/twSDK/twSDK-Core.js
// @grant        none
// ==/UserScript==

(function() {
    'use strict';
    // aguardar readiness do SDK
    const wait = setInterval(() => {
        if (window.TWSDK && window.TWSDK.isReady && window.TWSDK.isReady()) {
            clearInterval(wait);
            // seu código aqui
        }
    }, 200);
})();
```

## Uso por módulo - Snippets rápidos

Helpers (`twSDK-Helpers.js`)

### Propósito

Fornece funções utilitárias essenciais para acesso seguro a dados do jogo (`game_data`), extração de valores aninhados, limpeza e parsing de números, além de wrappers centralizados de logging. Deve ser carregado antes de qualquer outro módulo do SDK, pois provê utilidades básicas usadas por todos os demais módulos.

### Principais funções

- `window.TWSDK_Helpers.getGameData()` — Obtém a instância segura de `game_data`, priorizando `unsafeWindow` em userscripts.
- `window.TWSDK_Helpers.safeGet(obj, path, defaultValue)` — Acessa valores aninhados de objetos via notação de ponto, retornando um valor padrão se o caminho não existir.
- `window.TWSDK_Helpers.cleanNumber(value)` — Remove caracteres não numéricos e converte para inteiro.
- `window.TWSDK_Helpers.log(level, ...args)` — Wrapper centralizado de logging, delega para `TWPROLogger` (quando disponível) ou para o `console`.
- `window.TWSDK_Helpers.info/warn/error/debug(...args)` — Atalhos para logging em diferentes níveis.

### Dependências

- Não possui dependências externas. Opcionalmente integra com `window.TWPROLogger` para logging avançado.

### Exemplos de uso

```js
// Acesso seguro ao game_data
const gd = window.TWSDK_Helpers.getGameData();
console.log('ID do jogador:', gd.player?.id);

// Extração segura de valores aninhados
const id = window.TWSDK_Helpers.safeGet(gd, 'player.id', 0);
console.log('ID seguro:', id);

// Limpeza de números
const n = window.TWSDK_Helpers.cleanNumber('1.234k');
console.log('Número limpo:', n);

// Logging centralizado
window.TWSDK_Helpers.info('Mensagem informativa');
window.TWSDK_Helpers.warn('Atenção!');
window.TWSDK_Helpers.error('Erro detectado');
window.TWSDK_Helpers.debug('Debug info:', {gd});
```

### Retornos

- `getGameData()`: objeto `game_data` do contexto do jogo, ou `{}` se indisponível.
- `safeGet(obj, path, defaultValue)`: valor encontrado no caminho informado ou o valor padrão.
- `cleanNumber(value)`: inteiro resultante da limpeza do valor informado.
- Métodos de logging não retornam valor (side-effect no console ou logger externo).

### Observações

- Expõe o objeto global `window.TWSDK_Helpers`.
- Deve ser carregado antes dos demais módulos do SDK.
- Utilizado internamente por todos os outros módulos para acesso seguro a dados e logging padronizado.
- Integra automaticamente com `TWPROLogger` se disponível, melhorando logs em ambientes compatíveis.

```js
// Exemplo rápido: logging customizado
window.TWSDK_Helpers.log('info', 'SDK carregado com sucesso');
```

Constants (`twSDK-Constants.js`)

### Propósito

Fornece um conjunto de constantes estáticas essenciais do Tribal Wars PRO, como pontos por nível de edifício, consumo de fazenda por unidade, produção base de recursos, multiplicadores de defesa e regex para coordenadas. Permite cálculos e validações sem depender de chamadas à API do jogo, garantindo performance e padronização.

### Principais objetos/propriedades

- `buildingPoints`: Pontos ganhos por nível de cada edifício (por tipo).
- `unitsFarmSpace`: Espaço de fazenda consumido por cada tipo de unidade.
- `resPerHour`: Produção de recursos por hora por nível (nível 0-30).
- `watchtowerLevels`: Multiplicador de defesa por nível da Torre de Vigia.
- `coordsRegex`: Expressão regular para detectar coordenadas no formato NNN|NNN.

### Exemplo de uso

```js
// Obter pontos por nível para um edifício
const points = window.TWSDK_Constants.buildingPoints['main'][5];
console.log('Pontos para main nível 5:', points);

// Espaço de fazenda consumido por uma unidade
const farmSpace = window.TWSDK_Constants.unitsFarmSpace['knight'];
console.log('Espaço de fazenda do cavaleiro:', farmSpace);

// Produção de recursos para nível 10
const prod = window.TWSDK_Constants.resPerHour[10];
console.log('Produção nível 10:', prod);

// Multiplicador de defesa da Torre de Vigia nível 5
const mult = window.TWSDK_Constants.watchtowerLevels[4];
console.log('Multiplicador Torre nível 5:', mult);

// Encontrar coordenadas em um texto
const coords = 'Aldeia em 123|456 e 321|654'.match(window.TWSDK_Constants.coordsRegex);
console.log('Coordenadas encontradas:', coords);
```

### Dependências

- Não possui dependências externas. Pode ser usado isoladamente ou em conjunto com outros módulos do SDK.

### Observações

- Expõe o objeto global `window.TWSDK_Constants`.
- Os dados são estáticos e raramente mudam, ideais para cálculos locais e validações rápidas.
- Útil para scripts que precisam de acesso rápido a parâmetros fixos do jogo.
- Carregado automaticamente ao importar o arquivo ou via `@require` em userscripts.

Utils (`twSDK-Utils.js`)

### Descrição

Fornece utilitários geográficos e de tempo para o Tribal Wars PRO, facilitando cálculos de distância, extração e manipulação de coordenadas, conversão de tempo e identificação de continentes. Expõe o objeto global `window.TWSDK_Utils`.

### Dependências

- [`twSDK-Constants.js`](twSDK-Constants.js:1) (usa `coordsRegex` para extração de coordenadas)

### Principais funções

- `window.TWSDK_Utils.calculateDistance(from, to)` — Calcula a distância exata em campos entre duas coordenadas no formato "X|Y".
- `window.TWSDK_Utils.getContinentByCoord(coord)` — Retorna o número do continente (K) com base em uma coordenada.
- `window.TWSDK_Utils.extractCoords(text)` — Extrai todas as coordenadas válidas encontradas em um texto.
- `window.TWSDK_Utils.getTravelTimeInSeconds(distance, unitSpeed)` — Calcula o tempo de viagem em segundos, dado a distância (campos) e a velocidade da unidade (min/campo).
- `window.TWSDK_Utils.secondsToHms(timestamp)` — Converte segundos totais para o formato HH:MM:SS.

### Exemplos de uso

```js
// Calcular distância entre duas coordenadas
const dist = window.TWSDK_Utils.calculateDistance('500|500', '505|505');
console.log('Distância:', dist); // 7.07

// Descobrir o continente de uma coordenada
const k = window.TWSDK_Utils.getContinentByCoord('550|600');
console.log('Continente:', k); // "65"

// Extrair todas as coordenadas de um texto
const coords = window.TWSDK_Utils.extractCoords('Aldeias: 123|456, 321|654');
console.log('Coordenadas encontradas:', coords); // ["123|456", "321|654"]

// Calcular tempo de viagem
const tempo = window.TWSDK_Utils.getTravelTimeInSeconds(15, 8);
console.log('Tempo de viagem (s):', tempo); // 7200

// Converter segundos para HH:MM:SS
const hms = window.TWSDK_Utils.secondsToHms(3661);
console.log('Tempo formatado:', hms); // "01:01:01"
```

### Observações

- Expõe o objeto global `window.TWSDK_Utils`.
- Utilitário independente, mas depende de `twSDK-Constants.js` para regex de coordenadas.
- Não realiza chamadas assíncronas; todas as funções são síncronas e rápidas.
- Ideal para scripts que precisam manipular coordenadas, calcular distâncias ou converter tempos no contexto do Tribal Wars PRO.

Player (`twSDK-Player.js`)

### Propósito

Fornece acesso estruturado e seguro aos dados do jogador atual, incluindo informações básicas (nome, ID, pontos, aldeias), rankings especializados (ODA, ODD, ODS), status de recursos Premium e recursos formatados para exibição. Facilita a integração de dados do jogador em scripts, UIs customizadas e automações.

### Dependências

- [`twSDK-Helpers.js`](twSDK/twSDK-Helpers.js:1)

### Principais funções

- `window.TWSDK_Player.getPlayer()` — Retorna objeto com dados completos do jogador (ID, nome, pontos, aldeias, tribo, status premium, etc).
- `window.TWSDK_Player.getSpecializedRanks()` — Retorna rankings especializados: ataque (ODA), defesa (ODD), suporte (ODS), e flag indicando se existem.
- `window.TWSDK_Player.getFeatures()` — Retorna status de recursos Premium, Farm Assistant e Gerente de Conta (ativos e possíveis).
- `window.TWSDK_Player.getPlayerInfo()` — Retorna objeto consolidado e formatado para exibição rápida na UI (nome, pontos, rankings, status premium, tribo, etc).

### Exemplos de uso

```js
// Dados básicos do jogador
const player = window.TWSDK_Player.getPlayer();
console.log('Nome:', player.name, 'Pontos:', player.points);

// Rankings especializados
const ranks = window.TWSDK_Player.getSpecializedRanks();
console.log('ODA:', ranks.rankAttack, 'ODD:', ranks.rankDefense, 'ODS:', ranks.rankSupport);

// Status de recursos Premium
const features = window.TWSDK_Player.getFeatures();
console.log('Premium ativo?', features.premium.active);

// Dados formatados para UI
const info = window.TWSDK_Player.getPlayerInfo();
console.log('Exibição rápida:', info);
```

### Retornos

- `getPlayer()`: `{ id, name, rank, points, villages, ally, allyLevel, allyMemberCount, isSitter, sleepStart, sleepEnd, dateStarted, isGuest, email_valid, incomings, supports, knightLocation, knightUnit, premiumPoints, rankFormatted, pointsFormatted }`
- `getSpecializedRanks()`: `{ rankAttack, rankDefense, rankSupport, hasSpecializedRanks }`
- `getFeatures()`: `{ premium: {possible, active}, farmAssistent: {possible, active}, accountManager: {possible, active} }`
- `getPlayerInfo()`: `{ id, name, rank, points, villages, rankFormatted, pointsFormatted, hasPremium, hasFarmAssistant, hasAccountManager, rankAttack, rankDefense, rankSupport, hasSpecializedRanks, ally, allyLevel, isInAlly }`

### Observações

- O módulo expõe `window.TWSDK_Player` para uso global e é agregado em `window.TWSDK` pelo Core.
- Depende de `window.TWSDK_Helpers` para acesso seguro a dados do jogo e limpeza de valores.
- Os retornos são objetos estruturados, prontos para integração com UI, automações ou lógica customizada.
- Os métodos não realizam chamadas assíncronas; todos os dados são extraídos do contexto atual do jogo (`game_data`).
- Para funcionamento correto, garanta que [`twSDK-Helpers.js`](twSDK/twSDK-Helpers.js:1) esteja carregado antes.

```js
// Exemplo rápido: nome do jogador formatado
const info = window.TWSDK_Player.getPlayerInfo();
console.log('Jogador:', info.name, '| Premium:', info.hasPremium);
```

Village (`twSDK-Village.js`)

### Propósito

Gerencia e fornece acesso estruturado aos dados da aldeia atual e múltiplas aldeias do jogador, incluindo recursos, edifícios, localização, bônus, população e integração com gerenciadores locais como o `TWPROVillageManager`. Permite obter informações detalhadas da aldeia ativa, listar todas as aldeias do jogador, acessar mapas indexados por ID e forçar atualização dos dados locais.

### Dependências

- [`twSDK-Helpers.js`](twSDK-Helpers.js:1) (obrigatório)
- Integração opcional com `window.TWPROVillageManager` (para múltiplas aldeias)

### Principais funções

- `window.TWSDK_Village.getVillage()` — Retorna objeto completo com dados da aldeia ativa: ID, nome, coordenadas, população, recursos, níveis de edifícios, bônus, pontos, status de upgradabilidade da fazenda, entre outros.
- `window.TWSDK_Village.getAllVillages()` — Retorna array com todas as aldeias do jogador (requer `TWPROVillageManager`).
- `window.TWSDK_Village.getAllVillagesMap()` — Retorna mapa `{ id: { name, coords } }` de todas as aldeias (requer `TWPROVillageManager`).
- `window.TWSDK_Village.refreshAllVillages()` — Força atualização dos dados de aldeias via `TWPROVillageManager`.

### Exemplos de uso

```js
// Obter dados completos da aldeia atual
const v = window.TWSDK_Village.getVillage();
console.log('Aldeia:', v.name, v.coord, '| População:', v.pop, '/', v.popMax);

// Listar todas as aldeias do jogador
const all = window.TWSDK_Village.getAllVillages();
all.forEach(vil => console.log(vil.id, vil.name, vil.coords));

// Obter mapa indexado por ID
const map = window.TWSDK_Village.getAllVillagesMap();
Object.entries(map).forEach(([id, info]) => console.log(id, info.name, info.coords));

// Forçar atualização das aldeias
window.TWSDK_Village.refreshAllVillages();
```

### Retornos

- `getVillage()`: `{ id, name, displayName, playerId, x, y, coord, pop, popMax, popPercent, wood, woodProd, stone, stoneProd, iron, ironProd, storageMax, buildings, buildingsLevel, bonus, bonusId, traderAway, modifications, points, lastResTick, isFarmUpgradable }`
- `getAllVillages()`: `[ { id, name, coords } ]` (requer `TWPROVillageManager` carregado)
- `getAllVillagesMap()`: `{ [id]: { name, coords } }` (requer `TWPROVillageManager` carregado)
- `refreshAllVillages()`: sem retorno (side-effect de atualização)

### Observações

- Expõe o objeto global `window.TWSDK_Village`.
- Para múltiplas aldeias, depende do carregamento do `TWPROVillageManager` no contexto do jogo.
- Os métodos retornam objetos estruturados, prontos para integração com UI, automações ou lógica customizada.
- Os dados da aldeia ativa são extraídos de `game_data` via helpers do SDK.
- Funções não realizam chamadas assíncronas; todas as operações são síncronas e rápidas.
- Integra automaticamente com `TWPROLogger` se disponível, melhorando logs em ambientes compatíveis.

```js
// Exemplo rápido: nome e coordenada da aldeia ativa
const v = window.TWSDK_Village.getVillage();
console.log('Aldeia:', v.name, '| Coord:', v.coord);
```

Combat (`twSDK-Combat.js`)

### Propósito

Fornece acesso estruturado a dados de unidades militares, composição de exércitos, defesa e fila de treinamento, facilitando cálculos, simulações e análises de combate no contexto do Tribal Wars PRO.

### Dependências

- [`twSDK-Helpers.js`](twSDK/twSDK-Helpers.js:1)

### Principais funções

- `window.TWSDK_Combat.getUnits()` — Obtém dados de todas as unidades militares disponíveis no mundo atual.
- `window.TWSDK_Combat.getUnitStats(unitId)` — Retorna os stats de uma unidade específica (ex: 'spear', 'knight').
- `window.TWSDK_Combat.getUnitsByType(type)` — Lista unidades agrupadas por tipo: 'infantry', 'cavalry', 'siege', 'special'.
- `window.TWSDK_Combat.getArmyComposition()` — Retorna a composição do exército presente na aldeia (quantidade por unidade e total).
- `window.TWSDK_Combat.getTrainingQueue()` — Obtém a fila de treinamento de tropas (ativa e pendentes).
- `window.TWSDK_Combat.getDefenses()` — Retorna composição defensiva da aldeia (unidades defensivas + nível do muro).

### Exemplo de uso

```js
// Obter todas as unidades disponíveis
const units = window.TWSDK_Combat.getUnits();
console.log('Unidades:', units);

// Stats de uma unidade específica
const stats = window.TWSDK_Combat.getUnitStats('spear');
console.log('Stats spear:', stats);

// Unidades por tipo
const infantaria = window.TWSDK_Combat.getUnitsByType('infantry');
console.log('Infantaria:', infantaria);

// Composição do exército da aldeia
const army = window.TWSDK_Combat.getArmyComposition();
console.log('Exército:', army);

// Fila de treinamento de tropas
const queue = window.TWSDK_Combat.getTrainingQueue();
console.log('Fila de treinamento:', queue);

// Composição defensiva
const defenses = window.TWSDK_Combat.getDefenses();
console.log('Defesas:', defenses);
```

### Observações

- O módulo expõe `window.TWSDK_Combat` para uso global.
- Depende de `window.TWSDK_Helpers` para acesso a dados do jogo e limpeza de valores.
- Os retornos são objetos estruturados, prontos para integração com UI ou lógica customizada.
- As funções utilizam dados de `game_data` e do contexto da aldeia atual.
- Para simulações de combate avançadas, utilize em conjunto com outros módulos do SDK.

Research (`twSDK-Research.js`)

### Descrição

Fornece acesso estruturado aos dados de pesquisas (tecnologias/ferrearia) e ao status do Paladino/Cavaleiro da aldeia atual. Permite consultar tecnologias disponíveis, fila de pesquisas, progresso da pesquisa em andamento e informações detalhadas do Paladino, facilitando automações, exibição de status e integrações com UIs customizadas.

### Dependências

- [`twSDK-Helpers.js`](twSDK/twSDK-Helpers.js:1)

### Principais funções

- `window.TWSDK_Research.getTechs()` — Retorna todas as tecnologias pesquisadas e disponíveis: `{ available, unavailable, full }`.
- `window.TWSDK_Research.getCurrentResearch()` — Retorna a pesquisa em andamento (unidade, duração, tempo restante, progresso) ou `null` se não houver.
- `window.TWSDK_Research.getResearchQueue()` — Retorna a fila de pesquisas pendentes (array de objetos com unidade, duração e posição).
- `window.TWSDK_Research.getTechStatus(unitId)` — Retorna o status de uma tecnologia específica ou `null`.
- `window.TWSDK_Research.getKnight()` — Retorna informações completas do Paladino/Cavaleiro (id, nome, nível, experiência, skills, status, etc).

### Exemplos de uso

```js
// Listar tecnologias pesquisadas e disponíveis
const techs = window.TWSDK_Research.getTechs();
console.log('Techs disponíveis:', techs.available);

// Verificar pesquisa em andamento
const current = window.TWSDK_Research.getCurrentResearch();
if (current) {
  console.log('Pesquisando:', current.unit, 'Progresso:', current.percentComplete + '%');
}

// Fila de pesquisas
const fila = window.TWSDK_Research.getResearchQueue();
console.log('Fila de pesquisas:', fila);

// Status de uma tecnologia específica
const status = window.TWSDK_Research.getTechStatus('axe');
console.log('Status do machado:', status);

// Dados do Paladino/Cavaleiro
const knight = window.TWSDK_Research.getKnight();
console.log('Paladino:', knight.name, '| Nível:', knight.level);
```

### Retornos

- `getTechs()`: `{ available: { [unitId]: { ... } }, unavailable: [unitId], full: { ... } }`
- `getCurrentResearch()`: `{ unit, duration, endDate, remainingTime, percentComplete } | null`
- `getResearchQueue()`: `[ { unit, duration, position } ]`
- `getTechStatus(unitId)`: objeto de status da tecnologia ou `null`
- `getKnight()`: `{ id, name, level, experience, experienceNextLevel, experiencePercent, skills: { offensive, defensive, mobility }, skillPoints, totalSkillPoints, isBerserk, isInjured, regimen, regimenBranch, full }`

### Observações

- O módulo expõe `window.TWSDK_Research` para uso global.
- Todos os métodos dependem do contexto do jogo (`game_data`).
- Os retornos são objetos estruturados, prontos para integração com UI ou automações.
- Para funcionamento correto, garanta que [`twSDK-Helpers.js`](twSDK/twSDK-Helpers.js:1) esteja carregado antes.
- O método `getKnight()` retorna valores zerados/campos nulos caso não haja Paladino ativo.

```js
// Exemplo rápido: listar skills do Paladino
const k = window.TWSDK_Research.getKnight();
console.log('Skills:', k.skills);
```

Buildings (`twSDK-Buildings.js`)

### Propósito

Gerencia a fila de construções da aldeia, permitindo acesso estruturado à construção ativa, pendentes e recomendações para próximos edifícios a serem construídos. Facilita a leitura do estado atual e a sugestão de upgrades, integrando-se ao contexto do jogo Tribal Wars PRO.

### Dependências

- [`twSDK-Helpers.js`](twSDK/twSDK-Helpers.js:1)

### Principais funções

- `window.TWSDK_Buildings.getBuildingQueue()` — Obtém a fila de construções ativa e pendente, além de status se a fila está cheia.
- `window.TWSDK_Buildings.getNextBuildings()` — Lista as próximas construções recomendadas, com custos, tempo e possibilidade de construção.

### Exemplo de uso

```js
// Obter a fila de construções
const queue = window.TWSDK_Buildings.getBuildingQueue();
console.log('Fila ativa:', queue.active);
console.log('Pendentes:', queue.pending);
console.log('Fila cheia?', queue.isFull);

// Obter recomendações de próximos edifícios
const next = window.TWSDK_Buildings.getNextBuildings();
console.log('Próximos edifícios:', next);
```

### Observações

- O módulo expõe `window.TWSDK_Buildings` para uso global.
- Depende de `window.TWSDK_Helpers` para acesso a dados e limpeza de valores.
- Os retornos são objetos estruturados, facilitando integração com UI ou lógica customizada.
- A fila de construções é baseada em `game_data.building_queue` e recomendações em `game_data.next_buildings`.
- Não há dependências externas além do Helpers.

Economy (`twSDK-Economy.js`)

### Descrição

Gerencia inventário de itens, efeitos ativos (como Premium e Farm Assistant), bônus diário, dados de farm (ataques automáticos) e quests do jogador. Permite acessar, listar e manipular informações essenciais para automação e análise de recursos, progresso e recompensas no Tribal Wars PRO.

### Dependências

- [`twSDK-Helpers.js`](twSDK/twSDK-Helpers.js:1)

### Principais funções

- `window.TWSDK_Economy.getItems()` — Retorna itens do inventário do jogador, separados em ativos e disponíveis.
- `window.TWSDK_Economy.getActiveEffects()` — Lista todos os efeitos ativos sobre o jogador (ex: Premium, Farm Assistant).
- `window.TWSDK_Economy.getDailyBonus()` — Obtém status e recompensas do bônus diário.
- `window.TWSDK_Economy.getFarmData()` — Retorna dados do farm: último ataque, recursos ganhos, estatísticas.
- `window.TWSDK_Economy.getQuests()` — Lista todas as quests (ativas, concluídas, disponíveis) e dados completos.
- `window.TWSDK_Economy.getQuestProgress(questId)` — Detalha o progresso de uma quest específica.
- `window.TWSDK_Economy.getQuestRewards()` — Lista recompensas disponíveis de quests.

### Exemplos de uso

```js
// Listar itens do inventário
const itens = window.TWSDK_Economy.getItems();
console.log('Itens ativos:', itens.active);
console.log('Itens disponíveis:', itens.available);

// Listar efeitos ativos
const efeitos = window.TWSDK_Economy.getActiveEffects();
console.log('Efeitos ativos:', efeitos);

// Consultar bônus diário
const bonus = window.TWSDK_Economy.getDailyBonus();
console.log('Bônus diário:', bonus);

// Dados de farm
const farm = window.TWSDK_Economy.getFarmData();
console.log('Farm:', farm);

// Listar quests
const quests = window.TWSDK_Economy.getQuests();
console.log('Quests ativas:', quests.active);

// Progresso de uma quest específica
const progresso = window.TWSDK_Economy.getQuestProgress('quest_id');
console.log('Progresso da quest:', progresso);

// Recompensas de quests
const recompensas = window.TWSDK_Economy.getQuestRewards();
console.log('Recompensas:', recompensas);
```

### Retornos

- `getItems()`: `{ active: [ {id, name, type, count, expiresAt, daysRemaining} ], available: [ {id, name, type, cost} ] }`
- `getActiveEffects()`: `[ {id, name, type, expiresAt, daysRemaining, isActive} ]`
- `getDailyBonus()`: `{ available, lastClaimedAt, nextAvailableAt, timesClaimedToday, maxClaimsPerDay, rewards: {wood, stone, iron, gold} }`
- `getFarmData()`: `{ hasAttack, lastAttack: {targetVillage, timestamp, resourcesGained, unitsLost, success}, statistics: {successfulAttacks, totalAttacks, totalResourcesGained} }`
- `getQuests()`: `{ active: [], completed: [], available: [], full }`
- `getQuestProgress(questId)`: `{ questId, goals: [ {id, type, progress, target, percent, completed} ], status, canClaim } | null`
- `getQuestRewards()`: `[ {questId, gold, woodChest, stoneChest, ironChest, premiumPoints, item} ]`

### Observações

- O módulo expõe `window.TWSDK_Economy` para uso global.
- Todos os métodos dependem do contexto do jogo (`game_data`).
- Retornos são objetos estruturados, prontos para integração com UI ou automações.
- Ideal para scripts que automatizam coleta de recompensas, análise de inventário ou monitoramento de progresso.
- Para funcionamento correto, garanta que [`twSDK-Helpers.js`](twSDK/twSDK-Helpers.js:1) esteja carregado antes.

```js
// Exemplo rápido: listar e exibir itens ativos
const ativos = window.TWSDK_Economy.getItems().active;
ativos.forEach(item => console.log(item.name, item.count));
```

Awards (`twSDK-Awards.js`)

Logger (`twSDK-Logger.js`)

### Descrição

Logger centralizado para o SDK e projetos derivados. Permite configurar nível mínimo de exibição, ativar/desativar logs, customizar prefixo, timestamps e até definir um handler customizado para processar as mensagens. Ideal para padronizar e controlar a saída de logs em scripts e ferramentas baseadas no TWSDK.

### Principais funções

- `window.TWSDK_Logger.setLevel(level)` — Define o nível mínimo de mensagens exibidas (`debug`, `info`, `warn`, `error`).
- `window.TWSDK_Logger.setEnabled(flag)` — Ativa/desativa completamente o logger.
- `window.TWSDK_Logger.enable()` / `disable()` — Atalhos para ativar/desativar.
- `window.TWSDK_Logger.setPrefix(prefix)` — Define o prefixo exibido antes das mensagens.
- `window.TWSDK_Logger.setTimestamp(flag)` — Ativa/desativa inclusão de timestamp ISO nas mensagens.
- `window.TWSDK_Logger.setHandler(fn)` — Define função customizada para processar logs (`fn(level, formatted, ...args)`).
- `window.TWSDK_Logger.log(level, ...args)` — Emite log em qualquer nível.
- `window.TWSDK_Logger.debug/info/warn/error(...args)` — Atalhos para cada nível de log.

### Configuração

Por padrão, o logger exibe todos os níveis a partir de `debug`, com prefixo `twSDK` e timestamp ativado. Pode ser customizado conforme necessidade do projeto:

```js
window.TWSDK_Logger.setLevel('warn'); // Apenas warnings e erros
window.TWSDK_Logger.setPrefix('MeuScript');
window.TWSDK_Logger.setTimestamp(false); // Sem timestamp
window.TWSDK_Logger.setHandler((level, msg, ...args) => {
  // Envia logs para servidor externo ou UI customizada
});
```

### Exemplos de uso

```js
window.TWSDK_Logger.info('Script iniciado');
window.TWSDK_Logger.warn('Atenção: configuração ausente');
window.TWSDK_Logger.error('Erro crítico', {code: 500});
window.TWSDK_Logger.debug('Debug detalhado', {dados});

// Log genérico
window.TWSDK_Logger.log('info', 'Mensagem customizada');

// Desativar logs temporariamente
window.TWSDK_Logger.disable();
window.TWSDK_Logger.enable();
```

### Retornos

- Métodos de log (`log`, `debug`, `info`, `warn`, `error`) não retornam valor (side-effect no console, GM_log ou handler customizado).
- Métodos de configuração retornam `undefined`.

### Observações

- O módulo expõe `window.TWSDK_Logger` para uso global.
- Permite integração com handlers externos (ex: envio para servidor, UI customizada).
- Compatível com ambientes Tampermonkey/Violentmonkey (`GM_log`) e console padrão.
- Utilizado internamente por outros módulos do SDK para logging padronizado.


### Propósito

Captura, processa e fornece acesso aos desafios/prêmios diários (Daily Awards) do Tribal Wars PRO, permitindo leitura do progresso diretamente do DOM ou via requisição ao backend do jogo.

### Dependências

- [`twSDK-Helpers.js`](twSDK/twSDK-Helpers.js:1)

### Principais funções

- `window.TWSDK_Awards.getDailyAwards()` — Extrai desafios do DOM da página de prêmios.
- `window.TWSDK_Awards.getDailyAwardProgress(title)` — Obtém progresso de um desafio específico pelo título (DOM).
- `window.TWSDK_Awards.fetchDailyAwardsFromServer()` — Busca desafios diretamente do servidor via fetch (Promise).
- `window.TWSDK_Awards.getDailyAwardsAsync()` — Obtém desafios: tenta DOM, senão busca do servidor (Promise).
- `window.TWSDK_Awards.getDailyAwardProgressAsync(title)` — Obtém progresso de um desafio específico (Promise).

### Exemplo de uso

```js
// Listar todos os desafios diários visíveis no DOM
const awards = window.TWSDK_Awards.getDailyAwards();
console.log('Awards:', awards);

// Buscar progresso de um desafio específico
const progresso = window.TWSDK_Awards.getDailyAwardProgress('Atacante');
console.log('Progresso:', progresso);

// Buscar desafios do servidor (Promise)
window.TWSDK_Awards.fetchDailyAwardsFromServer().then(awards => {
    console.log('Awards do servidor:', awards);
});

// Obter desafios de forma resiliente (DOM ou servidor)
window.TWSDK_Awards.getDailyAwardsAsync().then(awards => {
    console.log('Awards (async):', awards);
});

// Progresso assíncrono de um desafio
window.TWSDK_Awards.getDailyAwardProgressAsync('Defensor').then(prog => {
    console.log('Progresso:', prog);
});
```

### Observações

- O módulo expõe `window.TWSDK_Awards` para uso global.
- As funções síncronas dependem de estar na página de prêmios para acessar o DOM.
- As funções assíncronas permitem buscar dados mesmo fora da página de prêmios.
- O parsing depende da estrutura `.award-box` no DOM do jogo.
- Retornos seguem o padrão: `{title, current, target, percent, visualPercent, formatted, isComplete}`.
- Não há dependências externas além do Helpers.

WorldData (`twSDK-WorldData.js`)

### Propósito

Gerencia o download, normalização e cache local (IndexedDB) de grandes conjuntos de dados do mundo do Tribal Wars PRO: aldeias, jogadores, tribos e conquistas. Permite consultas rápidas e resilientes, mesmo offline, e reduz a carga de requisições ao servidor.

### Principais funções

- `window.TWSDK.worldData.get(entity, maxAgeHours = 1)` — Busca e retorna dados de uma entidade ('village', 'player', 'ally', 'conquer'). Usa cache local se válido, senão baixa do servidor com retry e backoff.
- `window.TWSDK.worldData.parseCSV(text)` — Converte texto CSV do jogo em array bidimensional.
- `window.TWSDK.worldData.formatData(data, entity)` — Estrutura arrays genéricos em objetos tipados por entidade.
- `window.TWSDK.worldData.saveToDB(dbName, table, keyId, data)` — Salva dados massivos no IndexedDB.
- `window.TWSDK.worldData.getFromDB(dbName, table)` — Recupera dados do IndexedDB.

### Configuração interna

O módulo mantém um pool de conexões IndexedDB para performance e expõe a configuração de cada entidade:

```js
window.TWSDK_WorldData.config = {
  village: { dbName: 'twpro_villages', table: 'villages', key: 'villageId', url: '/map/village.txt' },
  player: { dbName: 'twpro_players', table: 'players', key: 'playerId', url: '/map/player.txt' },
  ally: { dbName: 'twpro_tribes', table: 'tribes', key: 'tribeId', url: '/map/ally.txt' },
  conquer: { dbName: 'twpro_conquer', table: 'conquer', key: '', url: '/map/conquer_extended.txt' }
};
```

### Fluxo típico de uso

1. Chame `get(entity)` para buscar dados (ex: 'village'). O método verifica o cache local (IndexedDB) e só baixa do servidor se necessário.
2. Os dados baixados são normalizados e salvos automaticamente no IndexedDB.
3. Consultas subsequentes usam o cache até expirar (`maxAgeHours`).

#### Exemplo básico

```js
// Buscar todas as aldeias do cache (IndexedDB)
window.TWSDK.worldData.get('village').then(villages => {
  console.log('Total aldeias no servidor:', villages.length);
});
```

### Retornos

- Arrays de objetos estruturados por entidade:
  - `village`: `{ villageId, name, x, y, coord, playerId, points, type }`
  - `player`: `{ playerId, name, tribeId, villages, points, rank }`
  - `ally`: `{ tribeId, name, tag, members, villages, points, allPoints, rank }`
  - `conquer`: `{ villageId, timestamp, newPlayerId, oldPlayerId, oldTribeId, newTribeId, points }`

### Dependências

- IndexedDB (nativo do navegador)
- `fetch` (nativo)
- `window.TWSDK_Helpers` para logging e avisos (opcional)

### Observações

- O módulo expõe `window.TWSDK_WorldData` e é agregado em `window.TWSDK.worldData` pelo Core.
- Todas as operações principais são baseadas em Promises.
- O cache IndexedDB é limpo e reescrito a cada atualização para evitar dados obsoletos.
- O método `get` implementa retry automático com exponential backoff para resiliência a falhas de rede.
- Caso o fetch falhe, retorna dados antigos do cache se disponíveis.
- Não há dependências externas além do Helpers (para logging).

```js
// Exemplo: buscar jogadores
window.TWSDK.worldData.get('player').then(players => {
  console.log('Jogadores:', players.length);
});
```

// Consultar com filtro (exemplo de uso avançado)
// (Necessário implementar query customizada sobre os dados retornados)
window.TWSDK.worldData.get('village').then(villages => {
  const proximas = villages.filter(v => Math.abs(v.x - 100) < 10 && Math.abs(v.y - 200) < 10);
  console.log('Aldeias próximas:', proximas.length);
});
```

UI (`twSDK-UI.js`)

### Visão geral

O módulo UI fornece componentes visuais reutilizáveis e helpers para criar interfaces padronizadas no estilo TWPRO, facilitando a integração de widgets, pop-ups, seções, tabelas e barras de ação em scripts e ferramentas. Expõe o objeto global `window.TWSDK_UI`.

- **Principais recursos:**
  - Renderização de widgets fixos (caixas/box) e pop-ups flutuantes.
  - Criação de seções, tabelas e barras de ação customizadas.
  - Helpers para escape seguro de HTML, gerenciamento de listeners e layout de módulos.
  - Compatível com jQuery (requisito do jogo) e integração com Dialog nativo do Tribal Wars.

### Principais funções

- `window.TWSDK_UI.escapeHtml(str)` — Escapa caracteres HTML para prevenir XSS.
- `window.TWSDK_UI.getGlobalStyle()` — Retorna CSS base para widgets e tabelas.
- `window.TWSDK_UI.renderBoxWidget(config)` — Renderiza um widget fixo embutido na página.
- `window.TWSDK_UI.renderFixedWidget(config)` — Renderiza uma janela pop-up flutuante.
- `window.TWSDK_UI.createSection(title, content, options)` — Cria uma seção padronizada.
- `window.TWSDK_UI.createTable(columns, rows, options)` — Cria uma tabela estilizada.
- `window.TWSDK_UI.createActionBar(leftControls, rightButtons, options)` — Cria uma barra de ação com controles e botões.
- `window.TWSDK_UI.createToggleButton(moduleName, isRunning)` — Cria botão toggle padrão (iniciar/parar).
- `window.TWSDK_UI.createDialog(config)` — Cria um dialog modal padronizado (usa Dialog nativa ou fallback).
- `window.TWSDK_UI.createListenerManager(moduleName)` — Gerencia listeners para evitar memory leaks.
- `window.TWSDK_UI.createModuleLayout(config)` — Layout completo de módulo (header, seções, action bar).

### Exemplos de uso

```js
// Abrir um modal simples
window.TWSDK_UI.renderFixedWidget({
  title: 'Teste',
  body: 'Olá TWSDK!'
});

// Renderizar um widget fixo (box)
window.TWSDK_UI.renderBoxWidget({
  id: 'meu-widget',
  title: 'Meu Widget',
  body: '<b>Conteúdo customizado</b>'
});

// Criar uma tabela customizada
const htmlTabela = window.TWSDK_UI.createTable([
  { label: 'Nome', key: 'name' },
  { label: 'Valor', key: 'value', align: 'right' }
], [
  { name: 'Madeira', value: 1000 },
  { name: 'Ferro', value: 800 }
]);

// Criar uma barra de ação
const htmlBarra = window.TWSDK_UI.createActionBar([
  '<span>Controles à esquerda</span>'
], [
  { label: 'Salvar', id: 'btn_salvar' }
]);

// Layout completo de módulo
const htmlModulo = window.TWSDK_UI.createModuleLayout({
  icon: '⚙️',
  name: 'Exemplo',
  desc: 'Descrição do módulo',
  sections: [
    { title: 'Seção 1', content: 'Conteúdo 1' },
    { title: 'Seção 2', content: 'Conteúdo 2' }
  ],
  rightButtons: [ { label: 'Ação', id: 'btn_acao' } ]
});

// Gerenciar listeners
const lm = window.TWSDK_UI.createListenerManager('MeuMódulo');
const key = lm.store(document.body, 'click', () => alert('Clicou!'));
lm.cleanup(); // Remove todos os listeners
```

### Retornos

- `escapeHtml(str)`: string segura para HTML.
- `getGlobalStyle()`: string com CSS.
- `renderBoxWidget(config)`, `renderFixedWidget(config)`: sem retorno (side-effect na UI).
- `createSection(...)`, `createTable(...)`, `createActionBar(...)`, `createModuleLayout(...)`: string HTML pronta para uso/injeção.
- `createToggleButton(...)`: objeto de configuração de botão.
- `createDialog(config)`: objeto com métodos para exibir/fechar dialog ou side-effect na UI.
- `createListenerManager(moduleName)`: objeto `{ store, cleanup, count }` para gerenciar listeners.

### Dependências

- Requer jQuery disponível no contexto do jogo.
- Integra com Dialog nativo do Tribal Wars se disponível; caso contrário, faz fallback para pop-up customizado.
- Utiliza helpers do próprio SDK para logging (`H()`) e integração opcional com `TWPROLogger`.

### Observações

- O módulo expõe `window.TWSDK_UI` para uso global e é agregado em `window.TWSDK.ui` pelo Core.
- Todos os componentes seguem a estética TWPRO e são responsivos.
- Ideal para scripts que precisam criar interfaces customizadas, pop-ups, tabelas e controles interativos no Tribal Wars PRO.

```js
// Exemplo rápido: criar e exibir um dialog
window.TWSDK_UI.createDialog({
  title: 'Aviso',
  body: 'Operação concluída!'
});
```


Core (`twSDK-Core.js`)

### Propósito

O módulo Core é o agregador central do TWSDK. Ele deve ser carregado por último e é responsável por unir todos os módulos `TWSDK_*` em um único objeto global `window.TWSDK`. Além disso, fornece métodos globais utilitários para inicialização, verificação de readiness, debug e acesso consolidado aos dados do jogo e do jogador.

### Principais métodos globais

- `window.TWSDK.getWorldInfo()` — Retorna informações do mundo/servidor (world, market, screen, mode, csrf, version, locale, device, etc).
- `window.TWSDK.getAll()` — Compila e retorna todos os dados relevantes em um único objeto (player, ranks, features, village, world).
- `window.TWSDK.isReady()` — Verifica se o contexto do jogo está carregado e pronto para uso.
- `window.TWSDK.debug()` — Exibe toda a estrutura `game_data` no console para inspeção e debug.

### Agregação

- Todos os métodos dos módulos `TWSDK_Player`, `TWSDK_Village`, `TWSDK_Combat`, `TWSDK_Research`, `TWSDK_Buildings`, `TWSDK_Economy`, `TWSDK_Awards` são agregados diretamente em `window.TWSDK`.
- Os módulos `TWSDK_Constants`, `TWSDK_Utils`, `TWSDK_WorldData`, `TWSDK_UI` ficam disponíveis como sub-objetos: `window.TWSDK.constants`, `window.TWSDK.utils`, `window.TWSDK.worldData`, `window.TWSDK.ui`.
- Atalhos de utilitários (`Utils`) também são expostos no nível raiz para compatibilidade.

### Dependências

- Requer que todos os módulos auxiliares (`TWSDK_*`) estejam carregados previamente. A ordem correta é fundamental: Helpers → Constants → Utils → Player → Village → Combat → Research → Buildings → Economy → Awards → WorldData → UI → Core.
- Depende de `window.TWSDK_Helpers` para acesso seguro a dados do jogo.

### Exemplo de uso

```js
// Verificar readiness e usar entrypoints
if (window.TWSDK && window.TWSDK.isReady()) {
    const info = window.TWSDK.getWorldInfo();
    console.log('TWSDK pronto para', info.world, info.market);
    const all = window.TWSDK.getAll();
    console.log('Dados consolidados:', all);
    window.TWSDK.debug(); // Exibe game_data completo
}
```

### Observações

- O Core expõe o objeto global `window.TWSDK`.
- Emite warnings no console caso algum módulo obrigatório não esteja carregado.
- Não adiciona dependências externas e é compatível com vanilla JS.
- Para debugging avançado, utilize `window.TWSDK.debug()` e consulte o console.
- O carregamento correto dos módulos é essencial para o funcionamento pleno do SDK.

## Exemplos práticos (mini-casos de uso)

1) Script que lista aldeias próximas da aldeia atual:

```js
if (window.TWSDK && window.TWSDK.isReady()) {
    const v = window.TWSDK.getVillage();
    window.TWSDK.worldData.query('village', {near: v.coord, radius: 15})
        .then(list => {
            console.log('Aldeias num raio 15:', list.map(i => i.coord));
        });
}
```

2) Notificação quando um prédio completar construção:

```js
// Observador simples da fila de construção
setInterval(() => {
    const queue = window.TWSDK_Buildings.getQueue();
    if (queue && queue.length && queue[0].finishesAt < Date.now() + 1000*60) {
        window.TWSDK_UI.notify('Construção', 'Um prédio vai terminar em < 1min');
    }
}, 30*1000);
```

## Convenções e padrões adotados

- Cada módulo expõe um objeto global `window.TWSDK_<ModuleName>` para uso direto, enquanto [`twSDK-Core.js`](twSDK/twSDK-Core.js:1) agrega uma API unificada em `window.TWSDK`.
- Promise-based: operações assíncronas (fetch, IndexedDB) retornam Promises.
- Não há dependência de frameworks externos: código vanilla compatível com o contexto do navegador embutido no jogo.

## Desenvolvimento e testes

- Para desenvolver localmente, abra os arquivos em sua IDE e use o console do navegador no contexto do jogo (DevTools) para testar chamadas `window.TWSDK.*`.
- Para testar worldData e IndexedDB, rode os módulos dentro do jogo ou de um ambiente que exponha os mesmos endpoints e `game_data`.
- Use logs do módulo (`window.TWSDK_Logger` ou funções de debug) para diagnosticar chamadas e lifecycle do SDK.

## Performance e caching

- O módulo [`twSDK-WorldData.js`](twSDK/twSDK-WorldData.js:1) foi projetado para baixar grandes volumes e armazenar em IndexedDB; preferir consultas via `worldData` ao invés de múltiplos fetchs em tempo real.
- Evitar chamadas repetidas a funções que consultam dados remotos sem cache local.

## Contribuições

- Abra issues com descrição clara do problema ou feature desejada.
- Pull requests: siga o padrão de código (vanilla JS, comentários em Português/inglês onde necessário) e inclua testes manuais claros no PR.

## Histórico e versão

Mantenha um CHANGELOG.md separado para histórico de alterações relevantes. Use tags no repositório para marcar releases.

---

Arquivo atualizado: [`twSDK/README.md`](twSDK/README.md:1)
