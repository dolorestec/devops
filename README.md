# Dolorestec - Devops

**Descrição**

>Repositório de actions e workflows para o projeto Dolorestec.

**Sumário**

- [Buildx](#buildx)
  - [Build](#build)
  - [Create](#create)
- [Registry](#registry)
    - [Login](#login)
    - [Logout](#logout)
- [Environment](#environment)
    - [Load](#load)
- [Version](#version)
        
## Buildx
> Actions para o build de imagens docker.
### Build
**Descrição**
>Realiza o build e o push de uma imagem docker.
**Entradas**:
> Todas as entradas são opcionais, exceto `image`, `tag` e `context`.
- **image**: Nome da imagem.
    - requerido: true
- **tag**: Tag da imagem.
    - requerido: true
- **context**: Contexto do build.
    - requerido: true
- **dockerfile**: Dockerfile do build.
    - requerido: false
- **push**: Realiza o push da imagem.
    - requerido: false
    - default: `false`
- **pull**: Realiza o pull da imagem.
    - requerido: false
    - default: `false`
- **load**: Realiza o load da imagem.
    - requerido: false
    - default: `false`
- **no-cache**: Não utiliza o cache do build.
    - requerido: false
    - default: `false`
**Exemplo**:
```yaml
- name: Docker image build
  shell: bash
  env:
    command: |
      docker buildx build \
      --cache-from "type=registry,mode=max,ref=ghcr.io/${{ github.repository }}:${{ inputs.tag }}" \
      --cache-to "type=registry,mode=max,ref=ghcr.io/${{ github.repository }}:${{ inputs.tag }}" \
      --iidfile image_id.txt \
      --file ${{ inputs.dockerfile }} \
      --tag "${{ inputs.image }}:${{ inputs.tag }}" \
      $([ ${{ inputs.push }} == "true" ] && echo "--push") \
      $([ ${{ inputs.no-cache }} == "true" ] && echo "--no-cache") \
      $([ ${{ inputs.pull }} == "true" ] && echo "--pull") \
      $([ ${{ inputs.load }} == "true" ] && echo "--load") \
      ${{ inputs.context }}
  run: |
    echo "::group::Build image"
    ${{ env.command }}
    echo "::endgroup::"
```
### Create
**Descrição**
> Cria um builder para o docker.
**Entradas**:
> Todas as entradas são opcionais, exceto `name` e `driver`.
- **name**: Nome do builder.
    - requerido: true
- **driver**: Driver do builder.
    - requerido: true
- **flags**: Flags do buildkitd.
    - requerido: false
    - default: `--buildkitd-flags "--allow-insecure-entitlement security.insecure --allow-insecure-entitlement network.host"`
- **use**: Usa o builder criado.
    - requerido: false
    - default: `false`
**Exemplo**:
```yaml
- name: Create Buildx
  shell: bash        
  run: |
    echo "::group::Create Buildx"
    docker buildx create \
    --name ${{ inputs.name }} \
    --driver ${{ inputs.driver }} \
    ${{ inputs.flags }} \
    $( [[ -z "${{ inputs.use }}" ]] || echo "--use")
    echo "::endgroup::"
```
## Registry
> Actions para o login e logout no registry.
### Login
**Descrição**
 >Realiza o login no registry.
**Entradas**:
> Todas as entradas são requeridas.
- **registry**: Endereço do registry.
    - requerido: true
- **username**: Usuário do registry.
    - requerido: true
- **token**: Token do registry.
    - requerido: true
**Exemplo**:
```yaml
- name: Login to registry
  shell: bash
  run: |
    echo "::group::Docker registry login"
    echo "Name: ${{ inputs.registry }}"
    echo ${{ inputs.token }} | docker login ${{ inputs.registry }} -u ${{ inputs.username }} --password-stdin      
    echo "::endgroup::"
```
### Logout
**Descrição**
>Realiza o logout no registry.
**Entradas**:
> Todas as entradas são requeridas.
- **registry**: Endereço do registry.
    - requerido: true
**Exemplo**:
```yaml
- name: Logout to registry
  shell: bash
  run: |
    echo "::group::Docker registry login"
    echo "Name: ${{ inputs.registry }}"
    docker logout ${{ inputs.registry }}
    echo "::endgroup::"
```
### Environment
> Actions para o setup de ambiente.
#### Load
**Descrição**
>Carrega as variáveis de ambiente de um arquivo.
**Entradas**:
> Todas as entradas são requeridas.
- **file**: Arquivo com as variáveis de ambiente.
    - requerido: true
**Exemplo**:
```yaml
- name: Download variables
  uses: actions/download-artifact@v3
  with:
    name: ${{ inputs.env_file }} 
- name: Set variables
  shell: bash
  run: |
    echo "::group::Load Variables"
    cat ${{ inputs.env_file }} | xargs -I {} echo "{}" >> $GITHUB_ENV
    cat ${{ inputs.env_file }} | xargs -I {} echo "{}"
    echo "::endgroup::"
```
### Version
**Descrição**
>Cria uma tag com a versão do projeto.
**Entradas**:
> Todas as entradas são opcionais, exceto `suffix` , `major`, `minor` e `patch`.
- **branch**: 
    - description: "Branch name"
    - required: true
- **tag**: 
    - description: "Tag name"
    - required: true
- **version**: 
    - description: "Version name"
    - required: true
- **suffix**: 
    - description: "Suffix name"
    - required: false
- **major**: 
    - description: "Major version"
    - required: false
    - default: `false`
- **minor**: 
    - description: "Minor version"
    - required: false
    - default: `false`
- **patch**:
    - description: "Patch version"
    - required: false
    - default: `false`
**Saídas**:
- **version**: Versão do projeto.
- **tag**: Tag do projeto.
- **branch**: Branch do projeto.
**Exemplo**:
```yaml
- name: Versioning major
      shell: bash
      if: ${{ inputs.major == 'true' }}
      run: |
        echo "::group::Versioning major"
        echo "DLRS_NEXT_VERSION=$( echo ${{ env.DLRS_VERSION }} | awk -F. '{print $1+1".0.0"}')" >> $GITHUB_ENV
        echo "DLRS_NEXT_TAG=$( echo ${{ env.DLRS_VERSION }} | awk -F. '{print $1+1".0.0"}')-SNAPSHOT" >> $GITHUB_ENV
        if [[ ${{ env.DLRS_BRANCH }} == "main" ]]; then
            echo "DLRS_NEXT_BRANCH=hotfix/$( echo ${{ env.DLRS_VERSION }} | awk -F. '{print $1+1".0.0"}')" >> $GITHUB_ENV
          elif [[ ${{ env.DLRS_BRANCH }} == "develop" ]]; then
            echo "DLRS_NEXT_BRANCH=release/$( echo ${{ env.DLRS_VERSION }} | awk -F. '{print $1+1".0.0"}')" >> $GITHUB_ENV
        fi
        echo "::endgroup::"[
      env:
        DLRS_BRANCH: ${{ inputs.branch }}
        DLRS_TAG: ${{ inputs.tag }}
        DLRS_VERSION: ${{ inputs.version }}

    - name: Versioning minor
      shell: bash
      if: ${{ inputs.minor == 'true' }}
      env:
        DLRS_BRANCH: ${{ inputs.branch }}
        DLRS_TAG: ${{ inputs.tag }}
        DLRS_VERSION: ${{ inputs.version }}
      run: |
        echo "::group::Versioning minor"
        echo "DLRS_NEXT_VERSION=$( echo ${{ env.DLRS_VERSION }} | awk -F. '{print $1"."$2+1".0"}')"
        echo "DLRS_NEXT_TAG=$( echo ${{ env.DLRS_VERSION }} | awk -F. '{print $1"."$2+1".0"}')-$( echo ${{ env.DLRS_TAG }} | awk -F- '{print $2}' )"
        if [[ ${{ env.DLRS_BRANCH }} == "main" ]]; then
            echo "DLRS_NEXT_BRANCH=hotfix/$( echo ${{ env.DLRS_VERSION }} | awk -F. '{print $1"."$2+1".0"}')"
          elif [[ ${{ env.DLRS_BRANCH }} == "develop" ]]; then
            echo "DLRS_NEXT_BRANCH=release/$( echo ${{ env.DLRS_VERSION }} | awk -F. '{print $1"."$2+1".0"}')"
        fi
        echo "::endgroup::"

    - name: Versioning patch
      shell: bash
      if: ${{ inputs.patch == 'true' }}
      env:
        DLRS_BRANCH: ${{ inputs.branch }}
        DLRS_TAG: ${{ inputs.tag }}
        DLRS_VERSION: ${{ inputs.version }}
      run: |
        echo "::group::Versioning patch"
        echo "DLRS_NEXT_VERSION=$( echo ${{ env.DLRS_VERSION }} | awk -F. '{print $1"."$2"."$3+1}')"
        echo "DLRS_NEXT_TAG=$( echo ${{ env.DLRS_VERSION }} | awk -F. '{print $1"."$2"."$3+1}')-$( echo ${{ env.DLRS_TAG }} | awk -F- '{print $2}' )"
        if [[ ${{ env.DLRS_BRANCH }} == "main" ]]; then
            echo "DLRS_NEXT_BRANCH=hotfix/$( echo ${{ env.DLRS_VERSION }} | awk -F. '{print $1"."$2"."$3+1}')"
          elif [[ ${{ env.DLRS_BRANCH }} == "develop" ]]; then
            echo "DLRS_NEXT_BRANCH=release/$( echo ${{ env.DLRS_VERSION }} | awk -F. '{print $1"."$2"."$3+1}')"
        fi
        echo "::endgroup::"

    - name: Versioning suffix
      shell: bash
      if: ${{ inputs.patch == 'false' && inputs.minor == 'false' && inputs.major == 'false' }}
      run: |
        echo "::group::Versioning suffix"
        if [[ ${{ env.DLRS_SUFFIX }} == "SNAPSHOT" ]]; then
            echo "DLRS_NEXT_SUFFIX=RC1"
        else 
            echo "DLRS_NEXT_SUFFIX=$( echo ${{ env.DLRS_SUFFIX }} | awk -FRC '{print "RC"$2+1}')"
        fi
        echo "::endgroup::"
      env:
        DLRS_BRANCH: ${{ inputs.branch }}
        DLRS_TAG: ${{ inputs.tag }}
        DLRS_VERSION: ${{ inputs.version }}
        DLRS_SUFFIX: ${{ inputs.suffix }}

    - name: Set outputs
      id: set_outputs
      shell: bash
      run: |
        echo "::group::Versioning outputs"
        echo "version=${{ env.DLRS_NEXT_VERSION }}" >> $GITHUB_OUTPUT
        echo "tag=${{ env.DLRS_NEXT_TAG }}" >> $GITHUB_OUTPUT
        echo "branch=${{ env.DLRS_NEXT_BRANCH }}" >> $GITHUB_OUTPUT
        echo "suffix=${{ env.DLRS_NEXT_SUFFIX }}" >> $GITHUB_OUTPUT
        echo "::endgroup::"
      env:
        DLRS_NEXT_BRANCH: ${{ env.DLRS_NEXT_BRANCH }}
        DLRS_NEXT_TAG: ${{ env.DLRS_NEXT_TAG }}
        DLRS_NEXT_VERSION: ${{ env.DLRS_NEXT_VERSION }}
        DLRS_NEXT_SUFFIX: ${{ env.DLRS_NEXT_SUFFIX }}
```