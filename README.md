
# 🧾 Документация Payment API Warren

**Базовый URL:** `https://warren.su/api/`

---

## 🔐 Аутентификация

Все запросы (кроме `/health`) требуют заголовки:

| Заголовок | Описание | Пример |
|------------|-----------|---------|
| `X-Public-Key` | Публичный ключ партнёра | `pk_6345063f681afe76563fa28b51181b77cac88f8281a78a3fa95ac86c1bd2` |
| `X-Secret-Key` | Секретный ключ партнёра | `sk_ace9d631e75d259901ae39545d445a3709cf142fbf67d3d70c37c701f637` |

---

## 📊 ПОЛУЧЕНИЕ БАЛАНСА

**Эндпоинт:** `GET /balance`  
**Описание:** Получение текущего баланса партнёра в **USDT**.

### 🧩 Пример запроса

```bash
curl -X GET "https://warren.su/api/balance" \
  -H "X-Public-Key: ваш_публичный_ключ" \
  -H "X-Secret-Key: ваш_секретный_ключ"
```

### 💻 Пример на PHP

```php
function getBalance($publicKey, $secretKey) {
    $ch = curl_init();
    curl_setopt_array($ch, [
        CURLOPT_URL => 'https://warren.su/api/balance',
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_HTTPHEADER => [
            'X-Public-Key: ' . $publicKey,
            'X-Secret-Key: ' . $secretKey,
            'Content-Type: application/json'
        ]
    ]);

    $response = curl_exec($ch);
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);
    return json_decode($response, true);
}

// Использование
$balance = getBalance(
    'ваш_публичный_ключ',
    'ваш_секретный_ключ'
);
```

### ✅ Пример успешного ответа

```json
{
    "status": "success",
    "balance": {
        "balance_usdt": "150.500000",
        "deposit_usdt": "25.000000"
    }
}
```

**Описание полей:**

| Поле | Тип | Описание |
|------|-----|-----------|
| `balance_usdt` | string | Основной баланс в USDT (6 знаков) |
| `deposit_usdt` | string | Авансовый депозит в USDT (6 знаков) |

---

## 💳 СОЗДАНИЕ ТРАНЗАКЦИИ

**Эндпоинт:** `POST /transactions`  
**Описание:** Создание новой платёжной транзакции с генерацией QR-кода для СБП.

### 🧩 Параметры запроса

| Параметр | Тип | Обязательный | Описание | Пример |
|-----------|------|---------------|-----------|---------|
| `amount` | decimal | ✅ | Сумма платежа в рублях | `1000.00` |
| `currency` | string | ✅ | Всегда `"RUB"` | `"RUB"` |
| `internal_uuid` | string | ✅ | Уникальный ID заказа в вашей системе | `"order-12345"` |
| `client_id` | string | ✅ | ID клиента в вашей системе | `"client-001"` |
| `webhook_url` | string | ❌ | URL для получения вебхуков о статусе платежа | `"https://ваш-сайт.com/webhook"` |

### 🧠 Пример запроса

```bash
curl -X POST "https://warren.su/api/transactions" \
  -H "X-Public-Key: ваш_публичный_ключ" \
  -H "X-Secret-Key: ваш_секретный_ключ" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 1000.00,
    "currency": "RUB",
    "internal_uuid": "order-221121345",
    "client_id": "client-1",
    "webhook_url": "https://ваш-сайт.com/webhook"
  }'
```

### ✅ Пример успешного ответа

```json
{
    "status": "success",
    "transaction": {
        "transaction_uuid": "f0e97f24-f019-4ae8-a244-90fddb57ff11",
        "amount": "1000.00",
        "amount_usdt": "12.382368",
        "partner_amount": "870.00",
        "partner_amount_usd": "10.772660",
        "usdt_rate": "80.7600",
        "partner_fee_percent": "13.00",
        "status": "details_issued",
        "internal_uuid": "order-221121345",
        "order_number": "2025-1027-1359-40743",
        "sbp_qr_link": "https://app.wapiserv.qrm.ooo/media/qr_codes/qr_code_AD10005A7AIHIN2P971AKUSEBTSTUCLH.png",
        "sbp_payment_link": "https://qr.nspk.ru/AD10005A7AIHIN2P971AKUSEBTSTUCLH?type=02&bank=100000000042&sum=100000&cur=RUB&crc=E22B",
        "webhook_url": "https://ваш-сайт.com/webhook"
    }
}
```

**Описание полей ответа:**

| Поле | Тип | Описание |
|------|-----|-----------|
| `transaction_uuid` | string | Уникальный ID транзакции в системе Warren.SU |
| `amount` | string | Исходная сумма в RUB (2 знака) |
| `amount_usdt` | string | Сумма в USDT по курсу (6 знаков) |
| `partner_amount` | string | Сумма после вычета комиссии в RUB (2 знака) |
| `partner_amount_usd` | string | Сумма партнёра в USDT после комиссии (6 знаков) |
| `usdt_rate` | string | Курс USDT/RUB (4 знака) |
| `partner_fee_percent` | string | Комиссия партнёра % (2 знака) |
| `status` | string | Статус транзакции |
| `internal_uuid` | string | Ваш internal_uuid |
| `order_number` | string | Номер заказа в системе |
| `sbp_qr_link` | string | Ссылка на QR-код для оплаты |
| `sbp_payment_link` | string | Ссылка для оплаты через СБП |
| `webhook_url` | string | URL для вебхук-уведомлений |

### 💻 Пример на PHP

```php
function createTransaction($publicKey, $secretKey, $orderData) {
    $ch = curl_init();
    curl_setopt_array($ch, [
        CURLOPT_URL => 'https://warren.su/api/transactions',
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_POST => true,
        CURLOPT_POSTFIELDS => json_encode($orderData),
        CURLOPT_HTTPHEADER => [
            'X-Public-Key: ' . $publicKey,
            'X-Secret-Key: ' . $secretKey,
            'Content-Type: application/json'
        ]
    ]);

    $response = curl_exec($ch);
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);
    return json_decode($response, true);
}

// Использование
$transaction = createTransaction(
    'ваш_публичный_ключ',
    'ваш_секретный_ключ',
    [
        'amount' => 1000.00,
        'currency' => 'RUB',
        'internal_uuid' => 'order-' . time(),
        'client_id' => 'client-1',
        'webhook_url' => 'https://ваш-сайт.com/webhook'
    ]
);
```

---

## 🔍 ПРОВЕРКА СТАТУСА ТРАНЗАКЦИИ

**Эндпоинт:** `GET /status?internal_uuid={ваш_internal_uuid}`  
**Описание:** Получение текущего статуса и деталей транзакции.

### 🧩 Пример запроса

```bash
curl -X GET "https://warren.su/api/status?internal_uuid=order-221121345" \
  -H "X-Public-Key: ваш_публичный_ключ" \
  -H "X-Secret-Key: ваш_секретный_ключ"
```

### 💻 Пример на PHP

```php
function getTransactionStatus($publicKey, $secretKey, $internalUUID) {
    $ch = curl_init();
    curl_setopt_array($ch, [
        CURLOPT_URL => 'https://warren.su/api/status?internal_uuid=' . urlencode($internalUUID),
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_HTTPHEADER => [
            'X-Public-Key: ' . $publicKey,
            'X-Secret-Key: ' . $secretKey,
            'Content-Type: ' . $secretKey,
            'Content-Type: application/json'
        ]
    ]);

    $response = curl_exec($ch);
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);
    return json_decode($response, true);
}

// Использование
$status = getTransactionStatus(
    'ваш_публичный_ключ',
    'ваш_секретный_ключ',
    'order-221121345'
);
```

### ✅ Пример успешного ответа

```json
{
    "status": "success",
    "transaction": {
        "transaction_uuid": "f0e97f24-f019-4ae8-a244-90fddb57ff11",
        "amount": "1000.00",
        "partner_amount": "870.00",
        "partner_amount_usd": "10.772660",
        "usdt_rate": "80.7600",
        "status": "paid",
        "internal_uuid": "order-221121345",
        "client_id": "client-1",
        "sbp_payment_link": "https://qr.nspk.ru/AD10005A7AIHIN2P971AKUSEBTSTUCLH",
        "webhook_url": "https://ваш-сайт.com/webhook",
        "created_at": "2025-10-27 10:30:00",
        "paid_at": "2025-10-27 10:35:22"
    }
}
```

### 📋 Возможные статусы транзакций

| Статус | Описание |
|--------|-----------|
| `pending` | Транзакция создана, ожидает генерации QR |
| `details_issued` | QR-код сгенерирован, ожидает оплаты |
| `processing` | Платеж в обработке |
| `paid` | Транзакция успешно оплачена |
| `timeout` | Время оплаты истекло |
| `cancelled` | Транзакция отменена |

---

## 🩺 ПРОВЕРКА СОСТОЯНИЯ СЕРВЕРА

**Эндпоинт:** `GET /health`  
**Описание:** Проверка работоспособности API и подключения к базе данных. **Не требует аутентификации.**

### 🧩 Пример запроса

```bash
curl -X GET "https://warren.su/api/health"
```

### 💻 Пример на PHP

```php
function checkHealth() {
    $ch = curl_init();
    curl_setopt_array($ch, [
        CURLOPT_URL => 'https://warren.su/api/health',
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_TIMEOUT => 10
    ]);

    $response = curl_exec($ch);
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);

    if ($httpCode === 200) {
        return json_decode($response, true);
    }
    return ['status' => 'error', 'message' => 'Service unavailable'];
}

// Использование
$health = checkHealth();
```

### ✅ Пример успешного ответа

```json
{
    "status": "success",
    "message": "API is working",
    "database": "connected",
    "tables": {
        "partners": "exists",
        "transactions": "exists"
    },
    "timestamp": "2025-10-27T10:30:00+00:00",
    "environment": "production"
}
```

---

## 🔔 ВЕБХУК-УВЕДОМЛЕНИЯ О СМЕНЕ СТАТУСА

**URL вебхука:** Указывается в параметре `webhook_url` при создании транзакции  
**Метод:** `POST`  
**Content-Type:** `application/json`

### 📨 Формат вебхука при смене статуса

```json
{
    "amount": "5100.00",
    "status": "paid",
    "currency": "RUB",
    "usdt_rate": "81.0180",
    "amount_usdt": "62.948974",
    "operation_id": "9b57a944-001e-4a5e-ba70-4c4d6e6f9f21",
    "order_number": "2025-1028-1151-3551",
    "internal_uuid": "orders-40012",
    "partner_amount": "4845.00",
    "previous_status": "details_issued",
    "transaction_uuid": "7193803c-6029-4236-bcd1-23a9764e851e",
    "partner_amount_usd": "59.801526",
    "partner_fee_percent": "5.00"
}
```

### 📋 Описание полей вебхука

| Поле | Тип | Описание |
|------|-----|-----------|
| `amount` | string | Сумма платежа в RUB (2 знака) |
| `status` | string | Новый статус транзакции |
| `currency` | string | Валюта платежа (всегда "RUB") |
| `usdt_rate` | string | Курс USDT/RUB (4 знака) |
| `amount_usdt` | string | Сумма в USDT (6 знаков) |
| `operation_id` | string | ID операции в платежной системе |
| `order_number` | string | Номер заказа в системе Warren |
| `internal_uuid` | string | Ваш внутренний ID заказа |
| `partner_amount` | string | Сумма партнера в RUB после комиссии (2 знака) |
| `previous_status` | string | Предыдущий статус транзакции |
| `transaction_uuid` | string | Уникальный ID транзакции Warren |
| `partner_amount_usd` | string | Сумма партнера в USDT (6 знаков) |
| `partner_fee_percent` | string | Комиссия партнера % (2 знака) |

### 🔄 Сценарии отправки вебхуков

**1. Создание QR-кода → `details_issued`**
```json
{
    "amount": "1000.00",
    "status": "details_issued",
    "currency": "RUB",
    "usdt_rate": "80.7600",
    "amount_usdt": "12.382368",
    "operation_id": "abc123...",
    "order_number": "2025-1027-1359-40743",
    "internal_uuid": "order-221121345",
    "partner_amount": "870.00",
    "previous_status": "pending",
    "transaction_uuid": "f0e97f24-f019-4ae8-a244-90fddb57ff11",
    "partner_amount_usd": "10.772660",
    "partner_fee_percent": "13.00"
}
```

**2. Успешная оплата → `paid`**
```json
{
    "amount": "1000.00",
    "status": "paid",
    "currency": "RUB",
    "usdt_rate": "80.7600",
    "amount_usdt": "12.382368",
    "operation_id": "abc123...",
    "order_number": "2025-1027-1359-40743",
    "internal_uuid": "order-221121345",
    "partner_amount": "870.00",
    "previous_status": "details_issued",
    "transaction_uuid": "f0e97f24-f019-4ae8-a244-90fddb57ff11",
    "partner_amount_usd": "10.772660",
    "partner_fee_percent": "13.00"
}
```

**3. Просрочка платежа → `timeout`**
```json
{
    "amount": "1000.00",
    "status": "timeout",
    "currency": "RUB",
    "usdt_rate": "80.7600",
    "amount_usdt": "12.382368",
    "operation_id": "abc123...",
    "order_number": "2025-1027-1359-40743",
    "internal_uuid": "order-221121345",
    "partner_amount": "870.00",
    "previous_status": "details_issued",
    "transaction_uuid": "f0e97f24-f019-4ae8-a244-90fddb57ff11",
    "partner_amount_usd": "10.772660",
    "partner_fee_percent": "13.00"
}
```

### ✅ Ответ на вебхук

Ваш сервер должен возвращать HTTP 200 и JSON:

```json
{
    "status": "success",
    "received": true
}
```

### 🛡️ Гарантии доставки

- **Автоматические повторы:** 3 попытки с интервалом 5 минут
- **Очередь вебхуков:** Гарантированная доставка даже при временной недоступности вашего сервера
- **Логирование:** Все отправки логируются для отладки

### 🧪 Тестирование вебхука

```bash
curl -X POST "https://ваш-сайт.com/webhook" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": "1000.00",
    "status": "paid",
    "currency": "RUB",
    "usdt_rate": "80.7600",
    "amount_usdt": "12.382368",
    "operation_id": "abc123def456",
    "order_number": "2025-1027-1359-40743",
    "internal_uuid": "order-221121345",
    "partner_amount": "870.00",
    "previous_status": "details_issued",
    "transaction_uuid": "f0e97f24-f019-4ae8-a244-90fddb57ff11",
    "partner_amount_usd": "10.772660",
    "partner_fee_percent": "13.00"
  }'
```

### 💻 Пример обработки вебхука на PHP

```php
// webhook_handler.php
header('Content-Type: application/json');

$input = json_decode(file_get_contents('php://input'), true);

if (!$input) {
    http_response_code(400);
    echo json_encode(['status' => 'error', 'message' => 'Invalid JSON']);
    exit;
}

// Валидация обязательных полей
$requiredFields = ['transaction_uuid', 'status', 'internal_uuid'];
foreach ($requiredFields as $field) {
    if (!isset($input[$field])) {
        http_response_code(400);
        echo json_encode(['status' => 'error', 'message' => "Missing field: $field"]);
        exit;
    }
}

try {
    // Обновляем статус заказа в вашей системе
    updateOrderStatus($input['internal_uuid'], $input['status'], $input);
    
    // Логируем получение вебхука
    logWebhook($input);
    
    // Возвращаем успешный ответ
    http_response_code(200);
    echo json_encode(['status' => 'success', 'received' => true]);
    
} catch (Exception $e) {
    http_response_code(500);
    echo json_encode(['status' => 'error', 'message' => 'Internal server error']);
}

function updateOrderStatus($internalUUID, $status, $webhookData) {
    // Ваша логика обновления статуса заказа
    // Например, обновление в базе данных
    
    $pdo = new PDO(...);
    $stmt = $pdo->prepare("
        UPDATE orders 
        SET status = :status, 
            updated_at = NOW(),
            transaction_data = :transaction_data
        WHERE internal_uuid = :internal_uuid
    ");
    
    $stmt->execute([
        'status' => $status,
        'internal_uuid' => $internalUUID,
        'transaction_data' => json_encode($webhookData)
    ]);
    
    // Дополнительные действия в зависимости от статуса
    if ($status === 'paid') {
        markOrderAsPaid($internalUUID);
    } elseif ($status === 'timeout') {
        markOrderAsExpired($internalUUID);
    }
}

function logWebhook($data) {
    file_put_contents(
        '/var/log/webhooks.log', 
        date('Y-m-d H:i:s') . ' - ' . json_encode($data) . "\n", 
        FILE_APPEND
    );
}
```

---

## ⚠️ ОБРАБОТКА ОШИБОК

### Коды HTTP статусов

| Код | Описание |
|------|-----------|
| 200 | Успешный запрос |
| 400 | Неверные параметры запроса |
| 401 | Ошибка аутентификации |
| 404 | Транзакция не найдена |
| 405 | Метод не поддерживается |
| 409 | Дублирующийся `internal_uuid` |
| 500 | Внутренняя ошибка сервера |

### 📋 Примеры ошибок

**Ошибка аутентификации:**
```json
{
    "status": "error",
    "message": "Invalid authentication credentials"
}
```

**Неверные параметры:**
```json
{
    "status": "error", 
    "message": "Missing required field: amount"
}
```

**Дубликат транзакции:**
```json
{
    "status": "error",
    "message": "Transaction with this internal_uuid already exists"
}
```

**Транзакция не найдена:**
```json
{
    "status": "error",
    "message": "Transaction not found"
}
```

---

## 🔄 ТИПОВОЙ РАБОЧИЙ ПРОЦЕСС

### 1. Создание заказа
```php
// Ваша система создает заказ
$orderData = [
    'amount' => 1500.00,
    'currency' => 'RUB', 
    'internal_uuid' => 'order-' . uniqid(),
    'client_id' => 'user-123',
    'webhook_url' => 'https://ваш-сайт.com/webhook'
];

// Вызов API Warren.SU
$result = createTransaction($publicKey, $secretKey, $orderData);
```

### 2. Обработка ответа
```php
if ($result['status'] === 'success') {
    $transaction = $result['transaction'];
    
    // Сохраняем в вашу БД
    saveToDatabase([
        'internal_uuid' => $orderData['internal_uuid'],
        'transaction_uuid' => $transaction['transaction_uuid'],
        'amount' => $transaction['amount'],
        'status' => $transaction['status'],
        'qr_link' => $transaction['sbp_qr_link'],
        'payment_link' => $transaction['sbp_payment_link'],
        'order_number' => $transaction['order_number'],
        'webhook_url' => $transaction['webhook_url']
    ]);
    
    // Показываем клиенту QR-код и ссылку для оплаты
    showPaymentToClient($transaction['sbp_qr_link'], $transaction['sbp_payment_link']);
}
```

### 3. Получение вебхука
```php
// Ваш сервер автоматически получит вебхук при изменении статуса платежа
// на указанный в webhook_url URL
function handleWebhook($webhookData) {
    // Обновляем статус заказа в вашей системе
    updateOrderStatus($webhookData['internal_uuid'], $webhookData['status']);
    
    // Возвращаем успешный ответ
    return ['status' => 'success', 'received' => true];
}
```

### 4. Резервная проверка статуса
```php
// Если вебхук не пришел, периодически проверяем статус
$status = getTransactionStatus($publicKey, $secretKey, $orderData['internal_uuid']);

if ($status['transaction']['status'] === 'paid') {
    completeOrder($orderData['internal_uuid']);
}
```

---

