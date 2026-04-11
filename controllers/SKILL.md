---
name: controllers
description: "Используй при разработке и рефакторинге ASP.NET Core контроллеров: контракт, маршруты, ответы, валидация и стиль кода."
---

# Controllers

## Цель
Использовать единый стандарт для контроллеров, который обеспечивает:
- чистый и поддерживаемый код без бизнес-логики в presentation слое;
- стабильный API-контракт и предсказуемое поведение endpoint;
- надежную обработку входа и ошибок;
- корректную документацию и тестируемость.

## Когда использовать
Используй этот skill, если нужно:
- создать новый ASP.NET Core контроллер;
- изменить endpoint, маршрут, DTO, коды ответа или атрибуты API;
- провести полный рефакторинг существующего контроллера;
- сделать PR-ревью контроллера.

## Связанные skills
- `.github/skills/controllers-contract-openapi/SKILL.md`
- `.github/skills/controllers-testing/SKILL.md`
- `.github/skills/validators/SKILL.md`

## Обязательные правила
1. Использовать primary constructor для внедрения зависимостей там, где это возможно.
2. Длина строки не должна превышать 120 символов.
3. Тексты логов и ошибок должны быть только на английском языке.
4. Контроллер не должен содержать бизнес-логику.
5. Всегда пробрасывать `CancellationToken` в нижние слои.
6. Для публичных endpoint возвращать корректные и предсказуемые HTTP-коды.
7. Любой breaking change выполнять только через согласованную migration strategy.
8. Для `200 OK` с телом обязательно указывать тип в `ProducesResponseType(typeof(...), 200)`.
9. Не использовать `Ok(new { ... })` в публичном API, выносить ответ в именованный DTO.
10. Для публичных методов добавлять XML `summary` на русском, `param` и `returns`.
11. Располагать action-методы в логическом порядке: сначала чтение (GET), затем изменения, сгруппированные по ресурсу/сценарию.

## Рекомендуемые практики
1. Использовать ресурсный стиль маршрутов.
2. Делать action-методы короткими: один метод = один сценарий.
3. Выносить повторяющийся код (route/body checks, mapping, client context) в переиспользуемые компоненты.
4. Разделять request и response DTO.
5. Использовать guard clauses вместо глубокой вложенности.
6. Явно документировать все возможные коды ответа через `ProducesResponseType`.
7. Поддерживать единый нейминг action-методов по намерению (`GetById`, `UpdateEmail`, `InitName`).
8. Проверять согласованность `route id` и `body.UserId`, если оба присутствуют.
9. Внутри группы изменений располагать методы по жизненному циклу сценария: create/update/state/remove и далее групповые операции.

## Анти-паттерны
1. Бизнес-правила и вычисления в контроллере.
2. Breaking-изменения без migration notes для клиентов.
3. `[ProducesResponseType(StatusCodes.Status200OK)]` без `typeof(...)` при ответе с телом.
4. Анонимные объекты в `Ok(new { ... })` вместо DTO.
5. Игнорирование `CancellationToken`.
6. Смешивание стилей маршрутов без причины.
7. Непредсказуемые коды ответа для одинаковых сценариев.
8. Дублирование validation/mapping логики в каждом методе.

## Примеры

### Do
```csharp
[HttpGet("{id:guid}")]
[ProducesResponseType(typeof(UserDto), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<IActionResult> GetById([FromRoute] Guid id, CancellationToken ct)
{
	User user = await usersRepository.GetByIdAsync(id, ct);
	return Ok(UserDto.Map(user));
}
```

### Don't
```csharp
[HttpGet]
[ProducesResponseType(StatusCodes.Status200OK)]
public async Task<IActionResult> GetById(Guid id)
{
	// Business logic inside controller
	var result = await service.CalculateAndPersistAsync(id);
	return Ok(new { result.Id, result.Name });
}
```

## Алгоритм принятия решений
1. Определи тип изменения: internal refactoring или контрактное изменение.
2. Если меняется маршрут, обязательные поля, типы или коды ответа, считай это breaking change.
3. Для breaking change подготовь migration strategy и обратную совместимость там, где это возможно.
4. Приведи endpoint к единым правилам: валидация входа, typed responses, predictable status codes.
5. Убери дублирование и бизнес-логику из controller layer.
6. Обнови OpenAPI и связанные тесты.
7. Проверь совместимость клиентов.

## Проверка перед завершением
1. Контроллер не содержит бизнес-логики.
2. Зависимости внедрены через primary constructor, где это возможно.
3. Нет строк длиннее 120 символов.
4. Логи и ошибки на английском.
5. Все `200 OK` с телом имеют типизированный `ProducesResponseType`.
6. Нет `Ok(new { ... })` в публичных endpoint.
7. Для breaking-изменений оформлена migration strategy.
8. Все публичные методы документированы XML-комментариями.
9. OpenAPI и тесты синхронизированы с итоговым контрактом.
10. Методы контроллера упорядочены логически и читаются сверху вниз по основным пользовательским сценариям.
