У меня есть медленный запрос.

`
public IQueryable<ContractForUnpaidInvoiceDto> GetContractForUnpaidInvoice(
     string merchantNumber)
 {
     bool isSqlServer = _context.Database.ProviderName == DatabaseProviderConstants.SqlServer;
     DateTimeOffset now = _clock.UtcNow();

     var firstPaymentSettlementInvoiceQuery = (
         from psi in _context.PaymentSettlementInvoice.AsNoTracking()
         join pc in _context.PartyContract.AsNoTracking() on psi.AccountHolderId equals pc.Id
         join merchantAh in _context.PartyAccountHolder.AsNoTracking() on pc.MerchantId equals merchantAh.RelationId
         join ah in _context.PartyAccountHolder.AsNoTracking() on pc.Id equals ah.Id
         where merchantAh.AccountNumber == merchantNumber
               && AccountHolderStatusConstants.AccountHolderStatusForUnpaidInvoice.Contains(ah.Status)
         group psi by psi.AccountHolderId into g
         select new
         {
             AccountHolderId = g.Key,
             LatestInvoiceId = g.Min(psi => psi.Id)
         });

     return _context.PartyContract
         .AsNoTracking()
         .Join(_context.PartyAccountHolder,
             x => x.Id,
             pah => pah.Id,
             (x, pah) => new { PartyContract = x, PartyAccountHolder = pah })
         .Join(_context.PartyAccountHolder,
             x => x.PartyContract.MerchantId,
             ahm => ahm.RelationId,
             (x, ahm) => new { x.PartyContract, x.PartyAccountHolder, MerchantAccountHolder = ahm })
         .Join(_context.PartyInvoice,
             x => x.PartyAccountHolder.InvoiceId,
             i => i.Id,
             (x, i) => new { x.PartyContract, x.PartyAccountHolder, x.MerchantAccountHolder, PartyInvoice = i })
         .Join(_context.PartyOrder,
             x => x.PartyInvoice.OrderId,
             o => o.Id,
             (x, o) => new { x.PartyContract, x.PartyAccountHolder, x.MerchantAccountHolder, x.PartyInvoice, PartyOrder = o })
         .Join(_context.PartySingleInvoice,
             x => x.PartyAccountHolder.Id,
             si => si.Id,
             (x, si) => new { x.PartyContract, x.PartyAccountHolder, x.MerchantAccountHolder, x.PartyInvoice, x.PartyOrder, PartySingleInvoice = si })
         .Join(_context.PartyMerchantContractPlanXref,
             x => x.PartySingleInvoice.SingleInvoicePlanId,
             mcpx => mcpx.ContractPlanId,
             (x, mcpx) => new { x.PartyContract, x.PartyAccountHolder, x.MerchantAccountHolder, x.PartyInvoice, x.PartyOrder, PartyMerchantContractPlanXref = mcpx })
         .Join(_context.PartyProductDefinition,
             x => x.PartyMerchantContractPlanXref.ProductDefinitionCode,
             pd => pd.Id,
             (x, pd) => new { x.PartyContract, x.PartyAccountHolder, x.MerchantAccountHolder, x.PartyInvoice, x.PartyOrder, PartyProductDefinition = pd })
         .Join(firstPaymentSettlementInvoiceQuery,
             x => x.PartyAccountHolder.Id,
             li => li.AccountHolderId,
             (x, li) => new { x.PartyContract, x.PartyAccountHolder, x.PartyInvoice, x.PartyOrder, x.PartyProductDefinition, x.MerchantAccountHolder, li.LatestInvoiceId })
         .Join(_context.PaymentSettlementInvoice,
             x => x.LatestInvoiceId,
             psi => psi.Id,
             (x, psi) => new { x.PartyContract, x.PartyAccountHolder, x.PartyInvoice, x.PartyOrder, x.PartyProductDefinition, x.MerchantAccountHolder, PaymentSettlementInvoice = psi })
         .Select(x => new ContractForUnpaidInvoiceDto
         {
             ContractNumber = x.PartyAccountHolder.AccountNumber,
             CustomerName = x.PartyContract.CustomerName,
             TgwPaymentMethod = x.PartyProductDefinition.TgwPaymentMethod,
             TransactionNumber = x.PartyOrder.TgwTransactionNumber,
             CustomerType = x.PartyProductDefinition.CustomerType,
             ProductCode = x.PartyProductDefinition.Id,
             InvoiceId = x.PartyInvoice.InvoiceNumber,
             InvoiceDate = x.PartyInvoice.InvoiceDate,
             Status = x.PartyAccountHolder.Status,
             AccountNumber = x.PartyAccountHolder.AccountNumber,
             CurrencyId = x.PartyAccountHolder.CurrencyId,
             AccountHolderId = x.PartyAccountHolder.Id,
             OverdueStartDate = x.PaymentSettlementInvoice != null ? x.PaymentSettlementInvoice.InvoiceDueDate : null,
             OverdueDays = 
                 isSqlServer
                     ? SqlServerDbFunctionsExtensions.DateDiffDay(
                         EF.Functions, x.PaymentSettlementInvoice!.InvoiceDueDate, now)
                     : MySqlDbFunctionsExtensions.DateDiffDay(
                         EF.Functions, x.PaymentSettlementInvoice!.InvoiceDueDate, now)
         });
 }
`

он генерирует sql query

`
-- -----------------------------------------------------------------------------
-- Запрос 1: Обзор контрактов мерчанта / расчётных счетов-фактур (LIMIT @p_limit)
-- Типы параметров из лога:
--   @now            DbType = DateTimeOffset
--   @merchantNumber Size = 45 (строка)
--   @p_limit        DbType = Int32
-- Значения, перехваченные в первом логе (SensitiveDataLogging ON):
--   @now            = '2026-06-09 13:28:40.7773560'
--   @merchantNumber = '100056777976'
--   @p_limit        = 11
-- -----------------------------------------------------------------------------
SET @now            = '2026-06-09 13:28:40.7773560';
SET @merchantNumber = '100056777976';
SET @p_limit        = 11;
 
SELECT `p0`.`account_holder_id`,
       `p0`.`account_number`,
       `p`.`customer_name`,
       CASE WHEN `p6`.`customer_type` = 0 THEN 0 ELSE 1 END,
       `p6`.`tgw_payment_method`,
       CAST(`p6`.`code` AS signed),
       CAST(`p3`.`tgw_transaction_number` AS char),
       `p2`.`invoice_number`,
       `p2`.`invoice_date`,
       CAST(`p0`.`status` AS signed),
       `p11`.`invoice_due_date`,
       CASE WHEN TIMESTAMPDIFF(DAY, `p11`.`invoice_due_date`, @now) > 0
            THEN TIMESTAMPDIFF(DAY, `p11`.`invoice_due_date`, @now)
            ELSE 0 END,
       `p0`.`currency_id`
FROM `party_contract` AS `p`
INNER JOIN `party_account_holder` AS `p0` ON `p`.`contract_id` = `p0`.`account_holder_id`
INNER JOIN `party_account_holder` AS `p1` ON `p`.`merchant_id` = `p1`.`relation_id`
INNER JOIN `party_invoice` AS `p2` ON `p0`.`invoice_id` = `p2`.`invoice_id`
INNER JOIN `party_order` AS `p3` ON `p2`.`order_id` = `p3`.`order_id`
INNER JOIN `party_single_invoice` AS `p4` ON `p0`.`account_holder_id` = `p4`.`single_invoice_id`
INNER JOIN `party_merchant_contract_plan_xref` AS `p5` ON `p4`.`single_invoice_plan_id` = `p5`.`contract_plan_id`
INNER JOIN `party_product_definition` AS `p6` ON CAST(`p5`.`product_definition_code` AS signed) = `p6`.`code`
INNER JOIN (
    SELECT `p7`.`account_holder_id` AS `AccountHolderId`,
           MIN(`p7`.`settlement_invoice_id`) AS `LatestInvoiceId`
    FROM `payment_settlement_invoice` AS `p7`
    INNER JOIN `party_contract` AS `p8` ON `p7`.`account_holder_id` = `p8`.`contract_id`
    INNER JOIN `party_account_holder` AS `p9` ON `p8`.`merchant_id` = `p9`.`relation_id`
    INNER JOIN `party_account_holder` AS `p10` ON `p8`.`contract_id` = `p10`.`account_holder_id`
    WHERE (`p9`.`account_number` = @merchantNumber)
      AND `p10`.`status` IN (0, 3, 4, 6)
    GROUP BY `p7`.`account_holder_id`
) AS `t` ON `p0`.`account_holder_id` = `t`.`AccountHolderId`
INNER JOIN `payment_settlement_invoice` AS `p11` ON `t`.`LatestInvoiceId` = `p11`.`settlement_invoice_id`
LIMIT @p_limit;
`

---

## Анализ медленного запроса `GetContractForUnpaidInvoice`

---

### Критические проблемы (убивают производительность)

#### 1. CAST в JOIN-условии — индекс не используется

```sql
INNER JOIN `party_product_definition` AS `p6`
    ON CAST(`p5`.`product_definition_code` AS signed) = `p6`.`code`
```

Тип `product_definition_code` в `party_merchant_contract_plan_xref` не совпадает с типом `code` в `party_product_definition`. MySQL вынужден делать CAST на каждой строке → индекс по `code` не используется → full scan.

**Фикс:** Исправить типы колонок в схеме так, чтобы они совпадали. Либо переопределить маппинг в EF Core:
```csharp
// В конфигурации сущности:
builder.Property(x => x.ProductDefinitionCode)
    .HasConversion<int>();  // или совпадающий тип
```

---

#### 2. Фильтр по `merchantNumber` только в подзапросе

Внешний запрос сканирует **все** строки `party_contract`, затем фильтрует через INNER JOIN к подзапросу. Нет раннего WHERE на `merchantNumber`. MySQL не всегда пушит предикат внутрь.

Структура сейчас:
```
ALL party_contract rows
  → JOIN party_account_holder × 2
  → JOIN party_invoice, party_order, party_single_invoice, ...
  → JOIN [subquery with merchant filter]  ← фильтрация ЗДЕСЬ
```

**Фикс:** Переписать так, чтобы субзапрос был первым, а к нему джойнились остальные таблицы. Или использовать `FromSqlInterpolated` с явным WHERE в начале.

---

#### 3. TIMESTAMPDIFF вычисляется дважды

```sql
CASE WHEN TIMESTAMPDIFF(DAY, `p11`.`invoice_due_date`, @now) > 0
     THEN TIMESTAMPDIFF(DAY, `p11`.`invoice_due_date`, @now)  -- повтор!
     ELSE 0 END
```

**Фикс в SQL:** заменить на `GREATEST(TIMESTAMPDIFF(DAY, ..., @now), 0)`.

В EF Core это задаётся через `EF.Functions` — нет прямого аналога `GREATEST`, но можно перейти на `FromSqlInterpolated` или добавить `DbFunction` маппинг.

---

### Значительные проблемы

#### 4. Подзапрос дублирует джойны внешнего запроса

Подзапрос `firstPaymentSettlementInvoiceQuery` снова джойнит `party_contract`, `party_account_holder` (merchant), `party_account_holder` (account holder) — те же самые таблицы, что и внешний запрос. MySQL материализует этот подзапрос как derived table и джойнит к нему.

**Фикс:** Вынести в CTE (в MySQL 8+ поддерживаются) — это явно описывает намерение и даёт оптимизатору больше свободы:

```sql
WITH merchant_latest_psi AS (
    SELECT p7.account_holder_id, MIN(p7.settlement_invoice_id) AS latest_invoice_id
    FROM payment_settlement_invoice p7
    INNER JOIN party_contract p8 ON p7.account_holder_id = p8.contract_id
    INNER JOIN party_account_holder p9 ON p8.merchant_id = p9.relation_id
    WHERE p9.account_number = @merchantNumber
      AND p8.contract_id IN (
          SELECT account_holder_id FROM party_account_holder
          WHERE status IN (0, 3, 4, 6)
      )
    GROUP BY p7.account_holder_id
)
SELECT ...
FROM merchant_latest_psi mlp
INNER JOIN party_contract p ON mlp.account_holder_id = p.contract_id
...
```

В EF Core для сложных запросов — переходить на `FromSqlInterpolated`.

---

#### 5. Множественные CAST в SELECT

```sql
CAST(`p6`.`customer_type` ...)
CAST(`p6`.`code` AS signed)
CAST(`p3`.`tgw_transaction_number` AS char)
CAST(`p0`.`status` AS signed)
```

Четыре CAST в каждой возвращаемой строке указывают на несоответствие типов в entity-маппинге. Нужно выровнять типы C# моделей с типами БД.

---

### Проблемы кода (качество)

#### 6. Лишняя null-проверка на INNER JOIN

```csharp
OverdueStartDate = x.PaymentSettlementInvoice != null
    ? x.PaymentSettlementInvoice.InvoiceDueDate
    : null,
```

`PaymentSettlementInvoice` джойнится через INNER JOIN — он никогда не будет `null`. Проверка бессмысленна.

---

#### 7. Смешанный стиль LINQ

Подзапрос написан в query syntax (`from ... join ... where ... group ... select`), основной — в method syntax (`.Join().Join().Select()`). Нужно выбрать один стиль.

---

#### 8. Имя переменной вводит в заблуждение

```csharp
var firstPaymentSettlementInvoiceQuery = ...
    LatestInvoiceId = g.Min(psi => psi.Id)
```

`Min(Id)` возвращает самый первый (наименьший) ID. Но имя переменной `firstPayment...` не очевидно описывает суть: «первый (самый старый) unpaid settlement invoice по каждому account holder». Переименовать, например, в `earliestUnpaidSettlementByAccountHolder`.

---

### Рекомендуемые индексы

Для данного запроса критично проверить наличие:

| Таблица | Колонка | Зачем |
|---|---|---|
| `party_account_holder` | `account_number` | фильтр по merchantNumber |
| `party_account_holder` | `relation_id` | JOIN к merchant |
| `party_account_holder` | `status` | WHERE status IN (...) |
| `payment_settlement_invoice` | `account_holder_id` | GROUP BY в подзапросе |
| `party_contract` | `merchant_id` | JOIN к merchant |

Проверить через `EXPLAIN`:
```sql
EXPLAIN SELECT ...  -- вставить сгенерированный SQL
```
Смотреть на `type = ALL` — это full scan, нужен индекс.

---

### Итоговый приоритет фиксов

| Приоритет | Проблема | Влияние |
|---|---|---|
| 1 | CAST в JOIN (`product_definition_code`) | критично — убирает возможность использовать индекс |
| 2 | Ранняя фильтрация по merchantNumber | критично — сокращает объём обрабатываемых строк |
| 3 | Индексы (проверить EXPLAIN) | критично — зависит от объёма данных |
| 4 | GREATEST вместо двойного TIMESTAMPDIFF | значимо |
| 5 | CTE вместо derived subquery | значимо при большом объёме PSI |
| 6 | Выравнивание типов (убрать CAST в SELECT) | среднее |
| 7 | Null-check на INNER JOIN | минорное |