
====================
openedx-translations
====================

.. contents::
   :local:
   :depth: 1

----

Enquadramento da Componente
---------------------------

O ``openedx-translations`` é o repositório central de ficheiros de tradução da plataforma Open edX adoptada pela **Direção-Geral da Educação (DGE)**. Agrega e sincroniza ficheiros de tradução de mais de 50 repositórios Open edX com a plataforma Transifex, implementando o standard `OEP-58`_.

O repositório situa-se na camada de **localização e internacionalização (i18n)** da arquitectura da solução DGE, suportando a distribuição de traduções em Português para todos os componentes da plataforma — LMS, CMS, Micro-Frontends (MFEs), XBlocks e aplicações móveis.

**Funcionalidades principais:**

- Extracção automática de ficheiros fonte em inglês a partir dos repositórios Open edX (via GitHub Action ``extract-translation-source-files.yml``)
- Sincronização bidirecional com o projecto Transifex ``open-edx/openedx-translations`` (upload de fontes e download de traduções)
- Validação de ficheiros de tradução com GNU gettext ``msgfmt``
- Suporte a branches de release (``open-release/<release>.master``) em paralelo com ``main``
- Scripts Python para gestão de recursos Transifex e sincronização de releases
- Consumível por outros projetos via CLI ``openedx-atlas``

**Identificação EA:** Componente de Localização/i18n — transversal ao LMS, CMS, MFEs e aplicações móveis Open edX.

Pré-requisitos
--------------

.. list-table::
   :header-rows: 1
   :widths: 30 70

   * - Dependência
     - Versão / Detalhe
   * - Python
     - ≥ 3.11
   * - GNU gettext
     - Qualquer versão recente (comando ``msgfmt`` necessário para validação local)
   * - pip
     - latest
   * - openedx-atlas (consumidor)
     - latest (para consumir traduções noutros projetos)
   * - Conta Transifex
     - Necessária apenas para operações de sincronização manual

**Variáveis de ambiente:**

.. list-table::
   :header-rows: 1
   :widths: 35 65

   * - Variável
     - Descrição
   * - ``TRANSIFEX_TOKEN``
     - Token de API Transifex — necessário para os scripts de manutenção e workflows CI. Gerido como GitHub Actions Secret; nunca deve ser commitado.
   * - ``TRANSIFEX_PROJECT_SLUG``
     - Slug do projecto Transifex (por omissão: ``openedx-translations``). Pode ser sobreposto na invocação ``make``.
   * - ``GITHUB_TOKEN``
     - Token GitHub — gerido automaticamente pelos GitHub Actions.

Instalação e Execução Local
---------------------------

**Clonar o repositório:**

.. code-block:: bash

   git clone <repository-url>
   cd openedx-translations

**Instalar dependências de tradução:**

.. code-block:: bash

   pip install -r requirements/translations.txt

**Instalar dependências de teste:**

.. code-block:: bash

   make test_requirements
   # equivalente a: pip install -r requirements/test.txt

**Validar ficheiros de tradução localmente:**

.. code-block:: bash

   make validate_translation_files

**Corrigir nomes de recursos Transifex (modo simulação):**

.. code-block:: bash

   make fix_transifex_resource_names_dry_run

**Retentar merge de PRs Transifex pendentes:**

.. code-block:: bash

   make retry_merge_transifex_bot_pull_requests

.. note::
   A maioria das operações de sincronização com o Transifex é automatizada pelos GitHub Actions. A execução local dos scripts de manutenção requer a variável ``TRANSIFEX_TOKEN`` configurada no ambiente.

Configuração
------------

**``transifex.yml``**

Ficheiro de configuração da integração Transifex. Define, para cada componente Open edX, o mapeamento entre os ficheiros fonte em inglês e as expressões de caminho para os ficheiros de tradução por idioma.

Formatos de ficheiro suportados:

.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - Formato
     - Utilização
   * - ``PO``
     - Componentes Python/Django (edx-platform, XBlocks, course-discovery, etc.)
   * - ``KEYVALUEJSON``
     - Micro-Frontends React (frontend-app-*)
   * - ``ANDROID``
     - Aplicação móvel Android (``strings.xml``)
   * - ``STRINGS``
     - Aplicação móvel iOS (``Localizable.strings``)
   * - ``YAML_GENERIC``
     - tutor-contrib-aspects

Todos os recursos usam ``mode: onlyreviewed`` — apenas traduções revistas são sincronizadas.

**``Makefile``**

.. list-table::
   :header-rows: 1
   :widths: 40 60

   * - Comando
     - Descrição
   * - ``make test``
     - Executa os testes dos scripts Python com cobertura
   * - ``make validate_translation_files``
     - Valida compilabilidade dos ficheiros PO com ``msgfmt``
   * - ``make fix_transifex_resource_names``
     - Corrige nomes de recursos no projecto Transifex
   * - ``make fix_transifex_resource_names_dry_run``
     - Simulação da correção de nomes (sem alterações)
   * - ``make retry_merge_transifex_bot_pull_requests``
     - Retenta merge de PRs Transifex bloqueados
   * - ``make upgrade``
     - Actualiza os ficheiros ``requirements/*.txt``

O ``TRANSIFEX_PROJECT_SLUG`` pode ser sobreposto em qualquer comando ``make``:

.. code-block:: bash

   make TRANSIFEX_PROJECT_SLUG='openedx-translations-redwood' fix_transifex_resource_names

Testes
------

Os testes cobrem os scripts Python de manutenção em ``scripts/tests/``.

.. code-block:: bash

   # Instalar dependências de teste
   make test_requirements

   # Executar testes com cobertura
   make test
   # equivalente a: pytest -v -s --cov=. --cov-report=term-missing --cov-report=html scripts/tests

**Ficheiros de teste:**

- ``scripts/tests/test_fix_transifex_resource_names.py``
- ``scripts/tests/test_release_project_sync.py``
- ``scripts/tests/test_validate_translation_files.py``

A validação de ficheiros de tradução é também executada automaticamente pelo workflow ``validate-translation-files.yml`` em cada PR, com os erros publicados como comentário no PR.

Build e Deployment
------------------

Este repositório não produz artefactos de build. O deployment de traduções é feito em dois sentidos:

**Receber traduções do Transifex:**

O GitHub Actions workflow ``extract-translation-source-files.yml`` extrai regularmente os ficheiros fonte em inglês dos repositórios Open edX e adiciona-os a este repositório. A app GitHub Transifex faz o upload automático para Transifex e, após revisão, o download das traduções de volta para este repositório via PR automático.

**Consumir traduções noutros projetos:**

Os ficheiros de tradução são acedidos via CLI ``openedx-atlas``:

.. code-block:: bash

   # Exemplo: descarregar traduções de edx-platform para Português
   atlas pull translations/edx-platform/conf/locale

**Workflows GitHub Actions:**

.. list-table::
   :header-rows: 1
   :widths: 50 50

   * - Workflow
     - Descrição
   * - ``extract-translation-source-files.yml``
     - Extracção automática de ficheiros fonte dos repos Open edX
   * - ``validate-translation-files.yml``
     - Validação com ``msgfmt`` em cada PR
   * - ``automerge-transifex-app-prs.yml``
     - Auto-merge de PRs do bot Transifex
   * - ``fix-transifex-resource-names.yml``
     - Correção automática de nomes de recursos Transifex
   * - ``release-project-sync.yml``
     - Sincronização de projectos de release Transifex
   * - ``python-tests.yml``
     - Execução de testes Python (pytest)
   * - ``upgrade-python-requirements.yml``
     - Actualização automática de dependências Python

Estrutura do Repositório
------------------------

.. code-block:: text

   openedx-translations/
   ├── translations/                          # Ficheiros de tradução por componente
   │   ├── edx-platform/                     # Plataforma principal (PO)
   │   ├── frontend-app-*/                   # Micro-Frontends React (JSON)
   │   ├── xblock-*/                         # XBlocks (PO)
   │   ├── openedx-app-android/              # App móvel Android
   │   ├── openedx-app-ios/                  # App móvel iOS
   │   └── ...                               # 50+ componentes
   ├── scripts/                              # Scripts Python de manutenção
   │   ├── fix_transifex_resource_names.py   # Correção de nomes Transifex
   │   ├── validate_translation_files.py     # Validação de ficheiros PO
   │   ├── release_project_sync.py           # Sincronização de releases
   │   └── tests/                            # Testes dos scripts
   ├── .github/workflows/                    # Automação GitHub Actions
   ├── requirements/                         # Dependências Python
   │   ├── translations.txt                  # Ferramentas de tradução (compilado)
   │   ├── test.txt                          # Dependências de teste (compilado)
   │   ├── translations.in                   # Fonte de requirements de tradução
   │   └── test.in                           # Fonte de requirements de teste
   ├── transifex.yml                         # Configuração Transifex (50+ componentes)
   ├── Makefile                              # Comandos de manutenção
   └── catalog-info.yaml                     # Backstage catalog metadata

**Convenções:**

- Traduções em PO: ``translations/<componente>/<pacote>/conf/locale/<lang>/``
- Traduções em JSON: ``translations/<componente>/src/i18n/messages/<lang>.json``
- Apenas traduções revistas (``mode: onlyreviewed``) são sincronizadas do Transifex

Contribuição e Governação do Código
------------------------------------

**Workflow de branches:**

- Branch principal: ``main`` — para a versão mais recente do Open edX (Tutor nightly, edx-platform master)
- Branches de release: ``open-release/<release-name>.master`` — uma por cada release Open edX (ex: ``open-release/redwood.master``)
- Pull requests obrigatórios para qualquer alteração; revisão necessária antes de merge
- PRs do bot Transifex são auto-merged se os testes passarem (via ``automerge-transifex-app-prs.yml``)

**Processo para novas releases:**

Para cada nova release Open edX, é criada uma branch ``open-release/<release>.master`` e um novo projecto Transifex correspondente. O ``TRANSIFEX_PROJECT_SLUG`` deve ser definido ao invocar os comandos ``make`` de manutenção para releases.

**Atualização de dependências Python:**

.. code-block:: bash

   make upgrade

Documentação Complementar
--------------------------

- `OEP-58 — Open edX Translations Standard <https://github.com/openedx/open-edx-proposals/pull/367>`_
- `openedx-atlas CLI <https://github.com/openedx/openedx-atlas>`_
- `Transifex Open edX Project <https://app.transifex.com/open-edx/openedx-translations/dashboard/>`_
- `edx-i18n-tools <https://github.com/openedx/i18n-tools>`_
- `GitHub Actions workflows <.github/workflows/>`_

Licenciamento e SBOM
--------------------

**Licença:** Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0) — ver ``LICENSE``.

.. list-table::
   :header-rows: 1
   :widths: 22 12 22 18 12 22

   * - Nome
     - Versão
     - Fornecedor
     - Tipo de Licença
     - Tipo
     - purl
   * - edx-i18n-tools
     - 1.6.3
     - Axim Collaborative
     - Apache-2.0
     - runtime (scripts)
     - ``pkg:pypi/edx-i18n-tools@1.6.3``
   * - transifex-python
     - 3.5.0
     - Transifex
     - Apache-2.0
     - runtime (scripts)
     - ``pkg:pypi/transifex-python@3.5.0``
   * - transifex-client
     - 0.12.5
     - Transifex
     - Apache-2.0
     - runtime (scripts)
     - ``pkg:pypi/transifex-client@0.12.5``
   * - requests
     - 2.32.3
     - Python Requests
     - Apache-2.0
     - runtime (scripts)
     - ``pkg:pypi/requests@2.32.3``
   * - pyyaml
     - 6.0.2
     - pyyaml team
     - MIT
     - runtime (scripts)
     - ``pkg:pypi/pyyaml@6.0.2``
   * - polib
     - 1.2.0
     - David Jean Louis
     - MIT
     - runtime (scripts)
     - ``pkg:pypi/polib@1.2.0``
   * - lxml
     - 5.3.0
     - lxml team
     - BSD-3-Clause
     - runtime (scripts)
     - ``pkg:pypi/lxml@5.3.0``
   * - python-slugify
     - 8.0.4
     - Val Neekman
     - MIT
     - runtime (scripts)
     - ``pkg:pypi/python-slugify@8.0.4``
   * - pytest
     - 8.3.2
     - pytest-dev
     - MIT
     - dev (testes)
     - ``pkg:pypi/pytest@8.3.2``
   * - pytest-cov
     - 5.0.0
     - pytest-cov team
     - MIT
     - dev (testes)
     - ``pkg:pypi/pytest-cov@5.0.0``
   * - responses
     - 0.25.3
     - Sentry team
     - Apache-2.0
     - dev (testes)
     - ``pkg:pypi/responses@0.25.3``

Segurança
---------

- **Gestão de segredos:** O repositório não contém passwords, API keys, tokens, certificados privados ou connection strings reais. O token Transifex (``TRANSIFEX_TOKEN``) e o GitHub token (``GITHUB_TOKEN``) são geridos exclusivamente como GitHub Actions Secrets.
- **Conteúdo do repositório:** Os ficheiros de tradução (PO, JSON, XML, YAML) contêm apenas texto localizado, sem dados sensíveis ou configurações de sistema.
- **Controlo de acesso:** A sincronização automática com Transifex requer permissões de escrita no repositório, configuradas nos GitHub Actions Secrets da organização.
- **Reporte de vulnerabilidades:** Reportar por contacto directo com os responsáveis técnicos do projecto.

Versão Entregue e Correspondência com o EA em Produção
------------------------------------------------------

.. list-table::
   :header-rows: 1
   :widths: 30 70

   * - Campo
     - Valor
   * - Versão
     - —
   * - Tag/Release
     - —
   * - Commit hash
     - ``7a01b4c``
   * - Data de release
     - *(a preencher)*
   * - Ambiente
     - —

----

.. rubric:: Developer Documentation (original below)

.. raw:: html

   <hr>

openedx-translations
####################

This openedx-translations repository contains translation files from Open edX repositories
to be kept in sync with Transifex. To accomplish this task, a GitHub Action in
``.github/workflows/`` named ``extract-translation-source-files.yml`` regularly extracts
English translation source files form Open edX repositories containing code and adds them
to this repository. A GitHub Transifex app allows for the automatic upload of these
translation files and after being translated on Transifex, the automatic download back
into this repository. The translation files in this repository can then be accessed by
using the `openedx-atlas`_ CLI tool to download specific directories of translation files
from openedx-translations.

This repository implements the `OEP-58`_ proposal.

Main and Release branches
*************************

This repository has a main branch in addition to a dedicated branch for every
release. As of May 10th, 2024 the following are the release branches:

``main`` branch
===============

This branch is used for the latest version of Open edX such as
`Tutor nightly`_, `edx-platform "master" branch`_ and others.

To translate the latest versions the `open-edx/openedx-translations`_ Transifex
project should be used.


``open-release/<release-name>.master`` branch
=============================================

This branch is used for the latest version of the Open edX Release, which will
be a version of Tutor and corresponding branches in tagged repos. For example,
for the Redwood release (June 2024), the branches were:
`Tutor Redwood v18`_, `edx-platform "open-release/redwood.master" branch`_
and others.

To update translations for a named release, find the corresponding named release project in the `Open EdX Transifex project <https://app.transifex.com/open-edx/>`_  by searching for the release name (for example, Redwood) in the search box.

Tools for repository maintainers
********************************

This repository contains both `GitHub Actions workflows`_ and
`Makefile programs`_ to automate and assist maintainers chores including:

Fix resource names in Transifex
===============================

The GitHub Transifex App integeration puts an inconvenient names for resources like ``translations..frontend-app-something..src-i18n-transifex-input--main``
instead of ``frontend-app-something``.

Running this command should be safe and can be ran multiple times on
both the main ``openedx-translations`` project or on release projects
by setting the ``TRANSIFEX_PROJECT_SLUG`` make variable as shown below::

    # Dry run the name fix
    make TRANSIFEX_PROJECT_SLUG='openedx-translations-zebrawood' fix_transifex_resource_names_dry_run
    # If runs without errors, run the actual command:
    make TRANSIFEX_PROJECT_SLUG='openedx-translations-zebrawood' fix_transifex_resource_names

Translation validation
======================

This repository validates translations with the GNU gettext ``msgfmt`` tool.

The validation can be run locally with the following command:

.. code-block:: bash

    make validate_translations


The validation errors is also posted as a comment on the update translation
pull requests.

Retry merging Transifex pull requests
====================================

If GitHub Actions has an outage or any other issues there will be a backlog
of stale unmerged Transifex bot pull requests. To re-run tests and merge the
pull requests, run the following command:

.. code-block:: bash

    make retry_merge_transifex_bot_pull_requests

.. _OEP-58: https://github.com/openedx/open-edx-proposals/pull/367
.. _openedx-atlas: https://github.com/openedx/openedx-atlas

.. _sync_translations.yml workflow on GitHub: https://github.com/openedx/openedx-translations/actions/workflows/sync-translations.yml

.. _open-edx/openedx-translations: https://app.transifex.com/open-edx/openedx-translations/dashboard/
.. _open-edx/openedx-translations-redwood: https://app.transifex.com/open-edx/openedx-translations-redwood/dashboard/


.. _Tutor nightly: https://docs.tutor.edly.io/tutorials/nightly.html
.. _edx-platform "master" branch: https://github.com/openedx/edx-platform
.. _Tutor Redwood v18: https://docs.tutor.edly.io/
.. _edx-platform "open-release/redwood.master" branch: https://github.com/openedx/edx-platform/tree/open-release/redwood.master

.. _GitHub Actions workflows: https://github.com/openedx/openedx-translations/tree/main/.github/workflows
.. _Makefile programs: https://github.com/openedx/openedx-translations/blob/main/Makefile
