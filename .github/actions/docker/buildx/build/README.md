# Buildx - Build

## Descrição 
Realiza o build e o push de uma imagem docker.

## Entradas:
Todas as entradas são opcionais, exceto `image`, `tag` e `context`.
### image: 
Nome da imagem.
- requerido: true
### tag
Tag da imagem.
- requerido: true
### context: 
Contexto do build.
- requerido: true
### dockerfile: 
Dockerfile do build.
- requerido: false
### push: 
Realiza o push da imagem.
- requerido: false
- default: `false`
### pull:
Realiza o pull da imagem.
- requerido: false
- default: `false`
### load: 
Realiza o load da imagem.
- requerido: false
- default: `false`
### no-cache**:
Não utiliza o cache do build.
- requerido: false
- default: `false`