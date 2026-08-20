# Affiliate Promo Decision Platform — Спецификация формул

Согласовано с заказчиком перед началом разработки. Каждое число на экранах приложения
должно быть прослеживаемо до одной из формул ниже — никаких "чёрных ящиков".

## 1. Базовые продажи и SKU mix

```
Baseline units (SKU, retailer) = Retailer monthly sales (категория) × SKU share × Кол-во месяцев в периоде
Eligible baseline = Σ Baseline units по выбранным retailers и SKU (после фильтров пользователя)
```

Все входные значения — sales по ритейлерам, доли SKU, список ритейлеров, период —
редактируются пользователем (п.3 задания).

## 2. Uplift и объём кампании

```
Incremental units = Eligible baseline × Sales Uplift %
Total promo units = Eligible baseline + Incremental units
```

**Sales Uplift % и Registration Rate % — редактируемые допущения (Base Demo Scenario:
15% / 30%), явно помечены в интерфейсе как "assumption", не как бенчмарк Samsung.**

## 3. Регистрации — два числа, не одно

```
Expected Registrations = Total promo units × Registration Rate %
```
— это спрос: сколько людей зарегистрировалось бы, если бы бюджет был не ограничен.

```
Max Fundable Registrations = Effective Funding Pool ÷ Average bonus per registration
```
— это предел: сколько регистраций реально можно оплатить при текущем бюджете.

Average bonus per registration взвешивается по SKU mix среди зарегистрировавшихся.

## 4. Bonus Liability — два разных числа, не путать

```
Potential Bonus Liability = Σ (Expected Registrations по SKU × Bonus per SKU)
```
Полная теоретическая нагрузка на бонусный бюджет, если бы деньги были не ограничены.
Бонус платится **всем** зарегистрировавшимся покупателям, включая тех, кто купил бы и без
акции (по чеку не видно, был ли покупатель "дополнительным") — это соответствует тому, как
реально работают affiliate-бонусы.

```
Effective Funding Pool = Samsung Promo Budget + (Partner Co-funding, если тумблер ON)
Actual / Funded Consumer Benefit Spend = MIN(Potential Bonus Liability; Effective Funding Pool)
Budget Coverage % = Funded Spend ÷ Potential Bonus Liability
```

## 5. Co-funding партнёра — отдельная строка учёта (по требованию заказчика)

Тумблер **"Учитывать co-funding партнёра"**:
- **OFF** — партнёр нигде не участвует в расчёте, всё считается от Samsung Promo Budget.
- **ON** — добавляется отдельная строка в бюджете:

```
Samsung Net Consumer-Benefit Cost = Funded Spend − Partner Co-funding
```

Наглядно показывает, как co-funding партнёра снижает нагрузку на бюджет Samsung
(в демо-примере: Max Fundable Registrations растёт с 172 до 241 при включении).

## 6. Итоговые метрики экрана решения

```
Total Campaign Investment = Samsung Net Consumer-Benefit Cost
                           + Digital Marketing + Offline Marketing
                           + Operation Cost + Other Costs

Cost per Registration (Expected) = Total Campaign Investment ÷ Expected Registrations
Cost per Registration (Funded)   = Total Campaign Investment ÷ Max Fundable Registrations

Cost per Incremental Unit = Total Campaign Investment ÷ Incremental units
```

**Показываем обе версии Cost per Registration рядом**, подписанные явно
("по спросу" / "по факту финансирования") — чтобы не путать гипотетическую и
реальную экономику.

## 7. Если бюджета не хватает — что предлагает система

Триггер: `Budget Coverage % < 100%`. Система показывает варианты (не автоматически
применяет, только предлагает):
- Снизить размер бонуса по SKU с наибольшей долей продаж
- Сузить список retailers / SKU (уменьшает Eligible baseline → меньше Potential Liability)
- Включить/увеличить partner co-funding
- Ввести лимит сертификатов явно (= Max Fundable Registrations, уже посчитан)
- Сократить период акции

## 8. Пример расчёта (Base Demo Scenario, все SKU и retailers, 1 месяц)

*Проверено кодом (`computeSimulation`, 11 автотестов), не ручным счётом — исходная версия
примера в чате содержала арифметическую ошибку в шаге 1 (было 5 600, верно — 5 000; Evrika
считается 0, пока маркетолог не введёт своё число). Формулы ниже — финальные, протестированные.*

| Метрика | Значение |
|---|---|
| Eligible baseline | 5 000 юнитов |
| Sales Uplift | 15% → Incremental 750 юнитов |
| Total promo units | 5 750 |
| Registration Rate | 30% |
| Expected Registrations | 1 725 |
| Potential Bonus Liability | 50 025 000 ₸ |
| Samsung Promo Budget | 5 000 000 ₸ |
| Partner Co-funding (ON) | 2 000 000 ₸ |
| Effective Funding Pool | 7 000 000 ₸ |
| Funded Consumer Benefit Spend | 7 000 000 ₸ |
| Budget Coverage | 14.0% |
| Samsung Net Cost | 5 000 000 ₸ |
| Max Fundable Registrations | ≈ 241 (co-funding OFF: ≈ 172) |
| Total Campaign Investment | 9 000 000 ₸ |
