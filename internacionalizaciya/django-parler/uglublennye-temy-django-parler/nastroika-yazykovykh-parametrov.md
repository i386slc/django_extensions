# Настройка языковых параметров

При необходимости проекты могут «форкнуть» языковые настройки парлера. Это редко требуется. Пример:

```python
from django.conf import settings
from parler import appsettings as parler_appsettings
from parler.utils import normalize_language_code, is_supported_django_language
from parler.utils.conf import add_default_language_settings

MYCMS_DEFAULT_LANGUAGE_CODE = getattr(
    settings, 'MYCMS_DEFAULT_LANGUAGE_CODE', FLUENT_DEFAULT_LANGUAGE_CODE
)
MYCMS_LANGUAGES = getattr(
    settings, 'MYCMS_LANGUAGES', parler_appsettings.PARLER_LANGUAGES
)

MYCMS_DEFAULT_LANGUAGE_CODE = normalize_language_code(MYCMS_DEFAULT_LANGUAGE_CODE)

MYCMS_LANGUAGES = add_default_language_settings(
    MYCMS_LANGUAGES, 'MYCMS_LANGUAGES',
    hide_untranslated=False,
    hide_untranslated_menu_items=False,
    code=MYCMS_DEFAULT_LANGUAGE_CODE,
    fallbacks=[MYCMS_DEFAULT_LANGUAGE_CODE]
)
```

Вместо использования функций из **parler.utils** (например, `get_active_language_choices()`) проект может получить доступ к настройкам языка, используя:

```python
MYCMS_LANGUAGES.get_language()
MYCMS_LANGUAGES.get_active_choices()
MYCMS_LANGUAGES.get_fallback_languages()
MYCMS_LANGUAGES.get_default_language()
MYCMS_LANGUAGES.get_first_language()
```

Эти методы добавляются функцией `add_default_language_settings()`. Подробности смотрите в классе **LanguagesSetting**.
