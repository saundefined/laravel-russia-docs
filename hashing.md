---
git: 46c2634ef5a4f15427c94a3157b626cf5bd3937f
---

# Хеширование

<a name="introduction"></a>
## Введение

[Фасад](/docs/{{version}}/facades) `Hash` фреймворка Laravel обеспечивает безопасное хеширование Bcrypt и Argon2 для хранения паролей пользователей. Если вы используете каркас одного из [стартовых комплектов приложений Laravel](starter-kits), то для регистрации и аутентификации по умолчанию будет использоваться Bcrypt.

Bcrypt – отличный выбор для хеширования паролей, потому что его «коэффициент работы» регулируется, а это означает, что время, необходимое для генерации хеш-кода, может быть увеличено по мере увеличения мощности оборудования. При хешировании паролей – чем медленнее, тем лучше. Чем больше времени требуется алгоритму для хеширования пароля, тем больше времени требуется злоумышленникам для создания «радужных таблиц» всех возможных строковых хеш-значений, которые могут использоваться в атаках.

<a name="configuration"></a>
## Конфигурирование

Драйвер хеширования по умолчанию для вашего приложения настраивается в файле конфигурации `config/hashing.php`. В настоящее время существует несколько поддерживаемых драйверов: [Bcrypt](https://en.wikipedia.org/wiki/Bcrypt) и [Argon2](https://en.wikipedia.org/wiki/Argon2) (вариации Argon2i и Argon2id).

<a name="basic-usage"></a>
## Основы использования

<a name="hashing-passwords"></a>
### Хеширование паролей

Вы можете хешировать пароль, вызвав метод `make` фасада `Hash`:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;

    class PasswordController extends Controller
    {
        /**
         * Обновить пароль пользователя.
         */
        public function update(Request $request): RedirectResponse
        {
            // Проверить длину нового пароля ...

            $request->user()->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();

            return redirect('/profile');
        }
    }

<a name="adjusting-the-bcrypt-work-factor"></a>
#### Регулировка коэффициента работы Bcrypt

Если вы используете алгоритм Bcrypt, метод `make` позволяет вам управлять коэффициентом работы алгоритма с помощью параметра `rounds`; однако значение по умолчанию приемлемо для большинства приложений:

    $hashed = Hash::make('password', [
        'rounds' => 12,
    ]);

<a name="adjusting-the-argon2-work-factor"></a>
#### Регулировка коэффициента работы Argon2

Если вы используете алгоритм Argon2, метод `make` позволяет вам управлять коэффициентом работы алгоритма с помощью параметров `memory`, `time` и `threads`; однако значения по умолчанию приемлемы для большинства приложений:

    $hashed = Hash::make('password', [
        'memory' => 1024,
        'time' => 2,
        'threads' => 2,
    ]);

> [!NOTE]    
> Дополнительную информацию об этих параметрах можно найти в [официальной документации PHP](https://www.php.net/manual/ru/function.password-hash.php).

<a name="verifying-that-a-password-matches-a-hash"></a>
### Проверка совпадения пароля с хешем

Метод `check` фасада `Hash` позволяет проверить, что указанная текстовая строка соответствует заданному хешу:

    if (Hash::check('plain-text', $hashedPassword)) {
        // Пароли совпадают ...
    }

<a name="determining-if-a-password-needs-to-be-rehashed"></a>
### Определение необходимости повторного хеширования пароля

Метод `needsRehash` фасада `Hash` позволяет определить, изменился ли коэффициентом работы, используемый хешером, с момента хеширования пароля. Некоторые приложения предпочитают выполнять эту проверку во время процесса аутентификации приложения:

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
