# WooCommerce Telegram Manager (n8n)

A Telegram-based, button-driven interface for checking WooCommerce orders, products, prices, stock status, and today's sales without opening the WordPress admin dashboard.

**Project status:** Read-only MVP with an experimental **Update Price** confirmation flow in **TEST MODE**. **No real price or inventory updates are implemented in this release.** The **Update Stock** menu item is a placeholder.

## Problem

Store operators often need quick answers about orders and products while away from their dashboards. Repeatedly opening WooCommerce for basic checks adds friction.

## Solution

An n8n workflow connects a Telegram bot to the WooCommerce API. Authorized users can select actions from an inline keyboard or use text commands. The workflow reads store data and returns results in Telegram.

```text
Telegram (messages / button callbacks)
            |
       n8n Trigger
            |
   User ID allowlist check
            |
    Menu / command routing
            |
  WooCommerce API (read only)
            |
     Telegram response
```

For multi-step prompts, the workflow uses an n8n Data Table to temporarily store per-user session state.

## Features

| Feature | Status |
| --- | --- |
| Inline Telegram main menu and Back to Menu navigation | Implemented |
| Telegram user ID allowlist | Implemented |
| Find an order by order ID | Implemented; verify against your store |
| Find a product by product ID | Implemented; verify against your store |
| Check product regular/current price | Implemented; verify against your store |
| Check stock management, quantity and status | Implemented; verify against your store |
| Today's order and sales summary | Implemented; verify against your store |
| Temporary user session and ID prompts | Implemented |
| Update Price | **TEST MODE only — no WooCommerce write** |
| Update Stock | **Not implemented** |

### Supported commands

```text
/start
/menu
/order 123
/product 123
/price 123
/stock 123
/sales
```

Replace `123` with a real order or product ID from your store.

## Requirements

- An n8n instance supporting the node types used in the export, including **Telegram Trigger**, **Telegram**, **WooCommerce**, **Data Table**, **Code**, **Switch**, and **Crypto**.
- A Telegram bot token created through BotFather.
- A WooCommerce store with its REST API enabled and appropriate API credentials.
- The Telegram numeric user ID(s) that should be permitted to use the bot.

The workflow was exported from an n8n 2.x environment. Compatibility with other versions has not been independently verified.

## Installation and setup

1. Import [`woocommerce-telegram-manager-sanitized.json`](./woocommerce-telegram-manager-sanitized.json) into n8n.
2. Reconnect **Telegram credentials** in the Telegram Trigger and all Telegram action nodes.
3. Reconnect **WooCommerce credentials** in the WooCommerce lookup nodes. For a read-only deployment, use an API key with **Read** permission.
4. Open **Config and Allowlist** and replace `REPLACE_WITH_YOUR_TELEGRAM_USER_ID` with the permitted numeric Telegram user ID. The existing check supports a comma-separated string of IDs.
5. Create an n8n Data Table (for example, `danira_telegram_sessions`) with these columns:

   | Column | Type |
   | --- | --- |
   | `user_id` | String |
   | `chat_id` | String |
   | `pending_action` | String |
   | `expires_at` | Number |
   | `product_id` | String |
   | `product_name` | String |
   | `original_price` | String |
   | `proposed_price` | String |
   | `token` | String |

6. Open **every Data Table node** and select the newly created table. The original table ID has been removed from the public export.
7. Confirm the **Telegram Trigger** receives both `message` and `callback_query` updates.
8. Save the workflow, test with a permitted Telegram account and then activate it when ready. Avoid testing with competing Telegram webhook subscriptions for the same bot.

**Note:** The export is intentionally inactive and has its environment-specific credential/table references removed. It is **not plug-and-play** until the above settings are restored.

## Verification checklist

Use your own test store or safe, real read-only records. Record results rather than assuming the workflow is production-ready.

- [ ] `/start` displays the main inline keyboard.
- [ ] An unauthorized Telegram user receives `Access denied`.
- [ ] Find Product returns a real product's ID, SKU, price and stock status.
- [ ] Find Order returns a real order's status and total.
- [ ] Check Price and Check Stock return data consistent with WooCommerce.
- [ ] Today's Sales returns a summary for the store's local day.
- [ ] Invalid or missing IDs produce an understandable response.
- [ ] Back to Menu works after successful results and errors.
- [ ] The Update Price test screen **does not modify** a product.
- [ ] The Update Stock button does not modify inventory.

### Test evidence

The repository does not currently include independently verified execution logs or screenshots. Add **redacted** Telegram screenshots or n8n execution evidence after completing the checks above. Do not publish customer information, Telegram identifiers, API keys or store-private data.

## Limitations

- **No write support:** The price confirmation path still says `TEST MODE: No price was changed.` There is no WooCommerce product-update operation in this export.
- **Stock updates are unavailable:** The Update Stock button leads to a coming-soon message.
- **The price flow is experimental:** Its session/confirmation handling should not be treated as proof of production-grade, concurrent, single-use authorization.
- **Product IDs:** The existing lookups are oriented around product IDs and do not implement a dedicated variation-management workflow.
- **Sales definition:** Today's sales summary counts `processing` and `completed` orders as paid; stores with different payment/status policies may need to adjust this definition.
- **Time and currency settings:** Store timezone and currency precision assumptions should be checked for the target WooCommerce store.
- **Permissions:** The allowlist is maintained inside an n8n configuration node. Configure it for your own users; do not publish real IDs.

For a strictly read-only public demo, hide the **Update Price** and **Update Stock** buttons (and their experimental paths) before enabling the workflow for other users.

## Security and responsible use

- Never commit Telegram bot tokens, WooCommerce API secrets, or real administrator IDs.
- Use the least-privileged WooCommerce API key suitable for this read-only workflow.
- Keep n8n access restricted and protect execution history, which may contain order/customer information.
- Review and sanitize screenshots and execution data before publishing.
- Do not enable writes by merely swapping the confirmation message; writes need verified access control, single-use confirmations, concurrency safety, API error handling and post-update verification.

## Repository files

```text
.
├── README.md
└── woocommerce-telegram-manager-sanitized.json
```

## Case study summary

**Problem →** Fast access to WooCommerce operational information is inconvenient from mobile.

**Workflow →** Telegram inline menu and commands, controlled through n8n.

**Integration →** Telegram Bot API, n8n, WooCommerce REST API and n8n Data Tables.

**Real test →** Perform and document the verification checklist against your own store.

**Result →** A reusable workflow for retrieving store information through an authorized Telegram bot, subject to your own successful integration tests.

**Limitations →** Read-only MVP; price confirmation is test-only; stock updates are not implemented.

**Reusable asset →** Sanitized n8n workflow JSON plus this setup README.

---

## Author

Developed by Zahra Raeisi  
AI & Automation Specialist  
Danira Automation
