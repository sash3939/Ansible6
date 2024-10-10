# Домашнее задание к занятию 6 «Создание собственных модулей»

## Подготовка к выполнению

1. Создайте пустой публичный репозиторий в своём любом проекте: `my_own_collection`.
2. Скачайте репозиторий Ansible: `git clone https://github.com/ansible/ansible.git` по любому, удобному вам пути.
3. Зайдите в директорию Ansible: `cd ansible`.
4. Создайте виртуальное окружение: `python3 -m venv venv`.
5. Активируйте виртуальное окружение: `. venv/bin/activate`. Дальнейшие действия производятся только в виртуальном окружении.
6. Установите зависимости `pip install -r requirements.txt`.
7. Запустите настройку окружения `. hacking/env-setup`.
8. Если все шаги прошли успешно — выйдите из виртуального окружения `deactivate`.
9. Ваше окружение настроено. Чтобы запустить его, нужно находиться в директории `ansible` и выполнить конструкцию `. venv/bin/activate && . hacking/env-setup`.

<img width="343" alt="Clone" src="https://github.com/user-attachments/assets/325125bc-fa93-413c-b8eb-f6d8fb419526">


## Основная часть

Ваша цель — написать собственный module, который вы можете использовать в своей role через playbook. Всё это должно быть собрано в виде collection и отправлено в ваш репозиторий.

**Шаг 1.** В виртуальном окружении создайте новый `my_own_module.py` файл.

**Шаг 2.** Наполните его содержимым:

```python
#!/usr/bin/python

# Copyright: (c) 2018, Terry Jones <terry.jones@example.org>
# GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)
from __future__ import (absolute_import, division, print_function)
__metaclass__ = type

DOCUMENTATION = r'''
---
module: my_test

short_description: This is my test module

# If this is part of a collection, you need to use semantic versioning,
# i.e. the version is of the form "2.5.0" and not "2.4".
version_added: "1.0.0"

description: This is my longer description explaining my test module.

options:
    name:
        description: This is the message to send to the test module.
        required: true
        type: str
    new:
        description:
            - Control to demo if the result of this module is changed or not.
            - Parameter description can be a list as well.
        required: false
        type: bool
# Specify this value according to your collection
# in format of namespace.collection.doc_fragment_name
extends_documentation_fragment:
    - my_namespace.my_collection.my_doc_fragment_name

author:
    - Your Name (@yourGitHubHandle)
'''

EXAMPLES = r'''
# Pass in a message
- name: Test with a message
  my_namespace.my_collection.my_test:
    name: hello world

# pass in a message and have changed true
- name: Test with a message and changed output
  my_namespace.my_collection.my_test:
    name: hello world
    new: true

# fail the module
- name: Test failure of the module
  my_namespace.my_collection.my_test:
    name: fail me
'''

RETURN = r'''
# These are examples of possible return values, and in general should use other names for return values.
original_message:
    description: The original name param that was passed in.
    type: str
    returned: always
    sample: 'hello world'
message:
    description: The output message that the test module generates.
    type: str
    returned: always
    sample: 'goodbye'
'''

from ansible.module_utils.basic import AnsibleModule


def run_module():
    # define available arguments/parameters a user can pass to the module
    module_args = dict(
        name=dict(type='str', required=True),
        new=dict(type='bool', required=False, default=False)
    )

    # seed the result dict in the object
    # we primarily care about changed and state
    # changed is if this module effectively modified the target
    # state will include any data that you want your module to pass back
    # for consumption, for example, in a subsequent task
    result = dict(
        changed=False,
        original_message='',
        message=''
    )

    # the AnsibleModule object will be our abstraction working with Ansible
    # this includes instantiation, a couple of common attr would be the
    # args/params passed to the execution, as well as if the module
    # supports check mode
    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )

    # if the user is working with this module in only check mode we do not
    # want to make any changes to the environment, just return the current
    # state with no modifications
    if module.check_mode:
        module.exit_json(**result)

    # manipulate or modify the state as needed (this is going to be the
    # part where your module will do what it needs to do)
    result['original_message'] = module.params['name']
    result['message'] = 'goodbye'

    # use whatever logic you need to determine whether or not this module
    # made any modifications to your target
    if module.params['new']:
        result['changed'] = True

    # during the execution of the module, if there is an exception or a
    # conditional state that effectively causes a failure, run
    # AnsibleModule.fail_json() to pass in the message and the result
    if module.params['name'] == 'fail me':
        module.fail_json(msg='You requested this to fail', **result)

    # in the event of a successful module execution, you will want to
    # simple AnsibleModule.exit_json(), passing the key/value results
    module.exit_json(**result)


def main():
    run_module()


if __name__ == '__main__':
    main()
```
Или возьмите это наполнение [из статьи](https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html#creating-a-module).

**Шаг 3.** Заполните файл в соответствии с требованиями Ansible так, чтобы он выполнял основную задачу: module должен создавать текстовый файл на удалённом хосте по пути, определённом в параметре `path`, с содержимым, определённым в параметре `content`.

<img width="568" alt="my_own_module" src="https://github.com/user-attachments/assets/453dc6f9-0c01-426b-aeb2-1ea49662a91f">

[my_own_module](https://github.com/sash3939/my_own_collection/blob/main/my_own_namespace/yandex_cloud_elk/plugins/modules/my_own_module.py)

**Шаг 4.** Проверьте module на исполняемость локально.

****1. Прежде чем проверять локально, нужно модуль (my_own_module) положить в папку ansible/lib/ansible/modules/

****2. Необходимо создать, например, playload.json в папке ansible и наполнить содержимым

<img width="511" alt="playload" src="https://github.com/user-attachments/assets/1a1abd79-d319-469e-8974-41dfb7708103">

<img width="838" alt="Local try" src="https://github.com/user-attachments/assets/9ea455ac-b63d-4841-a3f5-4e5fa119c54d">

<img width="367" alt="result" src="https://github.com/user-attachments/assets/61997f83-2138-4015-a0b7-20501a5f8b76">

**Шаг 5.** Напишите single task playbook и используйте module в нём.

<img width="470" alt="task" src="https://github.com/user-attachments/assets/3b866f6e-4274-400b-985c-4e3c2f77cb40">

**Шаг 6.** Проверьте через playbook на идемпотентность.

<img width="809" alt="playbook task" src="https://github.com/user-attachments/assets/8d8153a5-71a9-4cd5-844d-13c1ef2fcd0c">

**Шаг 7.** Выйдите из виртуального окружения.

**deactivate**

**Шаг 8.** Инициализируйте новую collection: `ansible-galaxy collection init my_own_namespace.yandex_cloud_elk`.

<img width="615" alt="create my_own_namespace" src="https://github.com/user-attachments/assets/871c4a9a-9990-43f1-96c7-917d83bf8fbe">

**Шаг 9.** В эту collection перенесите свой module в соответствующую директорию.

<img width="783" alt="copy module to collection" src="https://github.com/user-attachments/assets/d186a518-de78-49db-8f27-a325d83c2759">

**Шаг 10.** Single task playbook преобразуйте в single task role и перенесите в collection. У role должны быть default всех параметров module.

<img width="701" alt="single task role" src="https://github.com/user-attachments/assets/b7c169e2-2a95-4adb-a712-6847e8226b4c">

**Шаг 11.** Создайте playbook для использования этой role.

<img width="632" alt="role-defaults" src="https://github.com/user-attachments/assets/1f5c4e28-5607-47ca-ad54-dd076e7773ef">

<img width="613" alt="meta-roles" src="https://github.com/user-attachments/assets/599118de-ec2d-47f1-aaca-62885c867efd">

<img width="620" alt="role-tasks" src="https://github.com/user-attachments/assets/7f333811-bdf2-49cf-ba21-4b7941a2fd9e">

**Шаг 12.** Заполните всю документацию по collection, выложите в свой репозиторий, поставьте тег `1.0.0` на этот коммит.

[my_own_collection](https://github.com/sash3939/my_own_collection/tree/main)

[my_own_collection-tag](https://github.com/sash3939/my_own_collection/releases/tag/1.0.0)

**Шаг 13.** Создайте .tar.gz этой collection: `ansible-galaxy collection build` в корневой директории collection.

<img width="834" alt="tar gz collection" src="https://github.com/user-attachments/assets/a88b11e8-0ca5-4378-85c1-4559535d1601">

**Шаг 14.** Создайте ещё одну директорию любого наименования, перенесите туда single task playbook и архив c collection.

<img width="484" alt="test_collection" src="https://github.com/user-attachments/assets/bf4b1523-5728-4661-b30d-c9db3319f588">

**Шаг 15.** Установите collection из локального архива: `ansible-galaxy collection install <archivename>.tar.gz`.

<img width="760" alt="installed collection from local archive" src="https://github.com/user-attachments/assets/ec8f5be9-331b-40d7-85cb-28b81f45f8d9">

**Шаг 16.** Запустите playbook, убедитесь, что он работает.

<img width="391" alt="playbook local ready" src="https://github.com/user-attachments/assets/ee143a41-e9f9-46fc-ace6-839ceb9c92ba">

**Шаг 17.** В ответ необходимо прислать ссылки на collection и tar.gz архив, а также скриншоты выполнения пунктов 4, 6, 15 и 16.

[collection](https://github.com/sash3939/my_own_collection/tree/main/my_own_namespace/yandex_cloud_elk)

[tar.gz](https://github.com/sash3939/my_own_collection/blob/main/test_collection/my_own_namespace-yandex_cloud_elk-1.0.0.tar.gz)

[Шаг 4](<img width="511" alt="playload" src="https://github.com/user-attachments/assets/1a1abd79-d319-469e-8974-41dfb7708103">

<img width="838" alt="Local try" src="https://github.com/user-attachments/assets/9ea455ac-b63d-4841-a3f5-4e5fa119c54d">

<img width="367" alt="result" src="https://github.com/user-attachments/assets/61997f83-2138-4015-a0b7-20501a5f8b76">)
## Необязательная часть

1. Реализуйте свой модуль для создания хостов в Yandex Cloud.
2. Модуль может и должен иметь зависимость от `yc`, основной функционал: создание ВМ с нужным сайзингом на основе нужной ОС. Дополнительные модули по созданию кластеров ClickHouse, MySQL и прочего реализовывать не надо, достаточно простейшего создания ВМ.
3. Модуль может формировать динамическое inventory, но эта часть не является обязательной, достаточно, чтобы он делал хосты с указанной спецификацией в YAML.
4. Протестируйте модуль на идемпотентность, исполнимость. При успехе добавьте этот модуль в свою коллекцию.
5. Измените playbook так, чтобы он умел создавать инфраструктуру под inventory, а после устанавливал весь ваш стек Observability на нужные хосты и настраивал его.
6. В итоге ваша коллекция обязательно должна содержать: clickhouse-role (если есть своя), lighthouse-role, vector-role, два модуля: my_own_module и модуль управления Yandex Cloud хостами и playbook, который демонстрирует создание Observability стека.

---
