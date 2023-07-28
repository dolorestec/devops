# Buildx - Create
## Descrição
Cria a instancia do builder do docker.
## Entradas
Todas as entradas são opcionais, exceto `name` e `driver`.

### name
Nome do builder.
- requerido: true
### driver
Driver do builder.
- requerido: true
### flags
Flags do buildkitd.
- requerido: false
- default: `--buildkitd-flags "--allow-insecure-entitlement security.insecure --allow-insecure-entitlement network.host"`
### use
Usa o builder criado.
- requerido: false
- default: `true`