# Как создать токен GITHUB_TOKEN для агента

Агенту нужен **fine-grained personal access token** (классический PAT с `repo`-скопом тоже работает, но fine-grained безопаснее: права ограничены одним репозиторием).

## Создание (fine-grained)

1. GitHub → аватар в правом верхнем углу → **Settings**.
2. В левом меню внизу: **Developer settings** (для организации — *Organization settings → Developer settings*).
3. **Personal access tokens → Fine-grained tokens → Generate new token**.
4. Заполнить:
   - **Resource owner**: владелец репозитория (ваш аккаунт или организация).
   - **Repository access**: **Only select repositories** → выбрать нужный репозиторий (например, `tclitmaxai/qwen-agent`). Не выбирайте «All repositories» и не используйте вариант «Public repositories (read-only)» — read-only токен ничего не запишет.
   - **Permissions → Repository permissions**:
     | Право | Значение | Зачем |
     |---|---|---|
     | `Contents` | **Read and write** | push веток `agent/…` |
     | `Pull requests` | **Read and write** | открытие/обновление PR |
     | `Issues` | **Read and write** | комментарии, новые issues |
     | `Metadata` | **Read** | обязательно (нужно для любых запросов) |
     | `Codespaces` | **Read and write** | только если нужны проверки в Codespaces (`USE_CODESPACES = True`) |
     | `Workflows` | **Read and write** | только если агент может править `.github/workflows` (иначе push workflow-файлов не пройдёт) |
   - **Expiration**: любой (для Kaggle-экспериментов удобно 30–90 дней; token можно пересоздать).
5. **Generate token** → скопировать значение сразу: `github_pat_…` (повторно показать его нельзя).

## Куда положить

### Kaggle (основной сценарий ноутбука)
1. Ноутбук → **Add-ons → Secrets** (или **Add-ons → Manage secrets**).
2. Ключ: `GITHUB_TOKEN`, значение: скопированный токен.
3. Ноутбук сам прочитает его: `UserSecretsClient().get_secret("GITHUB_TOKEN")`.

### Локально / другой окружение
- Переменная окружения `GITHUB_TOKEN` (ноутбук покажет «GITHUB_TOKEN: есть/нет»).

## Проверка

Запустите в ноутбуке ячейку с `check_token_access()`: она **без побочных эффектов** (пустые POST-запросы и `git push --dry-run`) покажет, какие права реально есть у токена:

```
Токен создан аккаунтом: <login> | владелец репозитория: <owner>
  Issues: Read and write     -> ✅ есть
  Pull requests: Read and write -> ✅ есть
  Contents: Read and write   -> ✅ есть
```

Типичные ошибки:
- `❌ нет (HTTP 404/403)` — у fine-grained токена нехватка прав часто приходит как 404: проверьте, что нужный репозиторий выбран в токене и что права стоят **Read and write**, а не Read-only.
- Токен создан другим аккаунтом, чем владелец репозитория, и не имеет прав на репозиторий — запись не пройдёт.
- Для **публичного репозитория без записи** (`GITHUB_WRITE = False`, `GITHUB_PUSH = False`) токен не обязателен, но без него действует лимит 60 запросов/час.
