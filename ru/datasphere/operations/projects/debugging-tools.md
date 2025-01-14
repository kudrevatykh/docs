---
title: Как решить проблемы в проекте в {{ ml-platform-full-name }}
---

# Решение проблем

При возникновении ошибок в проектах {{ ml-platform-name }} может потребоваться сброс установленных пакетов и вычислений или изменение настроек проекта. Для выполнения большинства этих задач потребуется [роль](../../security/index.md#roles-list) не ниже `{{ roles-datasphere-project-developer }}`. Для изменения размера хранилища проекта необходима роль не ниже `{{ roles-datasphere-project-editor }}`.

Следующие действия могут помочь, если в проект не удается войти:

1. {% include [include](../../../_includes/datasphere/ui-find-project.md) %}
1. Нажмите **{{ ui-key.yc-ui-datasphere.common.more }}**.
1. В открывшемся списке нажмите **{{ ui-key.yc-ui-datasphere.project-page.debugging-tools-key-value }}** и выберите необходимую задачу:
   * **{{ ui-key.yc-ui-datasphere.common.factory-reset }}** — сброс состояния интерпретатора и удаление всех установленных библиотек. Код ноутбуков, вывод ячеек и все ресурсы проекта сохранятся.
   * **{{ ui-key.yc-ui-datasphere.project-page.debugging-tools.reset-jupyter-workspace.title }}** — закрытие всех открытых файлов и ноутбуков.
   * **{{ ui-key.yc-ui-datasphere.project-page.debugging-tools.clear-all-output.title }}** — очистка вывода всех ячеек открытых ноутбуков.
   * **{{ ui-key.yc-ui-datasphere.project-page.change-project-disk-size }}** — изменение размера хранилища проекта. Проект будет остановлен.
   * **{{ ui-key.yc-ui-datasphere.project-page.stop-ide-title }}** — остановка всех вычислений, закрытие открытых ноутбуков и выключение ВМ.
