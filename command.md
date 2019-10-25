# Les commandes

Les commandes sont des événements à publier sur le broker de messages. Ils sont écoutés par l'écosystème Moeaoo et interprétés pour réaliser les actions associées.

## Commandes disponibles sur votre environnement meaoo de dev

### <a name="reset"></a> Reset

> **Protocol** : MQTT  
> **Topic** : `project-65/prod/city/reset`  
> **QoS** : `0`  
> **Description** : permet de réinitialiser le score de l'agent

Payload : vide

### <a name="reset"></a> Téléporter l'agent

> **Protocol** : MQTT  
> **Topic** : `project-65/prod/user/path-to-target`  
> **QoS** : `0`  
> **Description** : téléporte l'agent à un endroit de la map. Permet de tester les cas que l'on souhaite

Payload :
```json
{
    "vehicle_type": "walk",
    "path": [
        [21, 5.6],
        [20.9, 5.6]
    ],
    "costs": [0.0, 0.0]
}
```

|champ|description|
|---|---|
|`vehicle_type`|Indique le moyen de transport avec lequel déplacer l'agent. Ce champ peut prendre n'importe quelle valeur spécifiée dans la colonne *code technique* des [moyens de transport](city.md#vehicle_type). Au final la valeur peut rester fixe car elle n'a pas d'incidence sur la téléportation |
|`path`|doit contenir impérativement un array de deux valeurs, chacune d'entre elle étant un array contenant une position x et y. Le premier array contient les coordonnées d'atterrissage de la téléportation et le second array contient l'endroit où téléporter l'agent (dans notre cas x = 20.9 et Y=5.6. Le second array contient la position de destination +/- 0.1 sur x ou sur y. C'est comme si on téléportait sur le premier point et que l'on glissait tout de suite sur le second |
|`cost`|doit impérativement contenir deux valeurs à 0 (donc à ne pas modifier) |

### <a name="circulation"></a> Changer les conditions de circulation

> **Protocol** : MQTT  
> **Topic** : `project-65/prod/city/morph/traffic_conditions`  
> **QoS** : `0`  
> **Description** : modifie les conditions de circulation dans la ville (n'affecte que le déplacement en voiture)

Payload :
```json
[
	{"road": "edge_67", "slowing_factor": 3},
	{"road": "edge_68", "slowing_factor": 3}
]
```

|champ|description|
|---|---|
|`road`|identifiant du segment de route sur lequel appliquer le niveau de fluidité souhaité du traffic|
|`slowing_factor`|facteur de fluidité de la voie de ciculation concernée. Plage de valeurs:[1,10], ce sont des entiers, 1 = traffic fluide, 10 = traffic 10x plus lent|

### <a name="fermerouvrirmetro"></a> Fermer/ouvrir une ligne de métro

> **Protocol** : MQTT  
> **Topic** : `project-65/prod/city/morph/lines_state`  
> **QoS** : `0`  
> **Description** : ferme ou ouvre une section de ligne de métro. Les lignes étant bi directionnelle, une fermeture = 1 sens de circulation. Il est donc possible de fermer seulement le sens de circulation A->B et pas B->A

Payload :
```json
[
  {"line": "edge_6", "state": "close"},
  {"line": "edge_12", "state": "close"}
]
```
- c'est un tableau de 0 ou N entrées

|champ|description|
|---|---|
|`line`|identifiant du segment de route sur lequel appliquer l'état indiqué dans "state"|
|`state`|close ou open|

### <a name="fermerouvrirmetro"></a> Fermer/ouvrir une route

> **Protocol** : MQTT  
> **Topic** : `project-65/prod/city/morph/roads_status`  
> **QoS** : `0`  
> **Description** : Description ; ferme ou ouvre une route pour un moyen de transport donné. Certains moyens de transport routiers étant bi directionnels, une fermeture = 1 sens de circulation. Il est donc possible de fermer seulement le sens de circulation A->B et pas B->A

Payload :
```json
[
  {
      "car": [
          {"road": "edge_54", "state": "close"}
      ]
  },
  {
      "bike": [
          {"road": "edge_12", "state": "close"}
      ]
  },
  {
      "walk": [
          {"road": "edge_32", "state": "close"}
      ]
  }
]
```

|champ|description|
|---|---|
|`car`|tableau regroupant les voies "open" et "close" pour les robotaxis|
|`bike`|tableau regroupant les voies "open" et "close" pour les vélos|
|`walk`|tableau regroupant les voies "open" et "close" pour les déplacements à pieds|
|`road`|identifiant de la voie de ciculation concernée telle que répertoriée dans l'api graph|
|`state`|état de la voie de ciculation concernée (`open` ou `close`)|

## Commandes disponibles sur l'environnement meaoo de démo

*Ces commandes sont les seules qui seront acceptées lors de la démo par l'écosystème*.

### <a name="move"></a> Déplacer l'agent

> **Protocol** : MQTT  
> **Topic** : `project-65/prod/user/path`  
> **QoS** : `0`  
> **Description** : déplace l'agent depuis sa position courante vers une cible, en utilisant le moyen de transport indiqué <u>et le chemin le plus rapide</u>.

Payload :
```json
{
  "vehicle_type": "subway",
  "target": {"x": 4.4, "y": 0.2}
}
```

|champ|description|
|---|---|
|`vehicle_type`|Indique le moyen de transport avec lequel déplacer l'agent. Ce champ peut prendre n'importe quelle valeur spécifiée dans la colonne *code technique* des [moyens de transport](city.md#vehicle_type) |
|`target`|Position cible vers laquelle déplacer l'agent (exprimée dans le [système de coordonnées de la MeaooCity](city.md#coord)). Si la cible n'est pas sur une route ou sur une ligne de métro, le système trouve de lui-même le carrefour ou la station de métro la plus proche et l'utilise comme cible.|

#### Raisons pour lesquelles la commande pourrait ne pas fonctionner

* La commande n'est pas envoyée sur le bon topic
* Le payload n'est pas bien formatté.
* Les [règles de changement de transport](city.md#vehicle_type) ne sont pas respectées.
* La destination est égale à la position courante.
* Aucun [chemin](graph.md) ne permet d'aller à la cible.

## Utilisables sur votre environnement de développement

Ces commandes vous sont fournies afin de faciliter vos développements et vos tests. Elles ont un impact direct sur l'écosystème et permettent de manipuler [le contexte](context.md), l'agent, ou le graphe par exemple. 
Lors de la démo finale, ou pendant vos tests sur la MeaooCity réelle, ces commandes ne seront pas acceptées et vous retourneront au mieux un message d'erreur, au pire rien.
