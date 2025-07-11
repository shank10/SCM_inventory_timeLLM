# Supply Chain Optimization API using TimeLLM

## Overview

This project stems from a hackathon hosted by Amazon, whose focus was mostly on dashboard and UI. So I discarded all the UI bit and instead have focused only on the core API, which were ignored by the hackathon.

The idea is to build a general-purpose API for **e-commerce supply chain optimization**, powered by a **Time Series Language Model (TimeLLM)**. It predicts product demand based on sales data and market trend analysis, evaluates inventory sufficiency, accounts for supplier constraints, and recommends when and how much to reorder.

Designed as a modular, composable backend, it provides one public endpoint (`/ask`) for integration with any system or interface, routing internally to specialized ML modules.

---

## 🚩 Problem Statement

E-commerce businesses face demand volatility, seasonal patterns, and supplier delays. Manual inventory planning results in either stockouts or overstocking, leading to revenue loss and operational inefficiencies.

This API automates:

* Trend Detection (e.g., upcoming spikes/dips in demand)

* Demand forecasting (weekly granularity)

* Inventory simulation over time

* Order timing recommendations across multiple suppliers with different constraints

* Risk flags (e.g., predicted stockout on Week 4)

**Primary Use-Case:**

The model should be fed 24 months data, otherwise it wouldn't be able to detect seasonality impact. Weekly demand forecast and trend triggers, for a quarter should matter more as our emphasis is inventory management. A prime question to answer for a model will be something like, given that the current month is July, the current suppliers of Product A, take between 5 to 10 days to deliver the product to the warehouse and given a higher demand for the product in Summer, do I have enough inventory over next 5 weeks? When should I place new orders with supplier1, who can supply 10 of Product A in 5 days with a delay likelihood of 0.75; when should I place an order with Supplier2, who can supply 25 products in 10 days with a delay likelihood of 0.5?

---

## 🎯 Goals and Success Metrics

| Goal                              | Metric                               |
| --------------------------------- | ------------------------------------ |
| Predict demand accurately         | <10% MAPE over 12-week forecast      |
| Recommend optimal reorder timings | <5% stockout incidents in simulation |
| Flexible question-answer API      | Handle 5+ question types via `/ask`  |
| Portable for GitHub and reuse     | Self-contained with synthetic data   |

---

## 🧑‍💻 Target Users

* Developers building SCM tools
* Data scientists evaluating LLM + forecasting
* Researchers modeling seasonality with suppliers

---

## 🔧 Features

* `/ask`: Unified endpoint for external queries
* Time-aware demand prediction (weekly, 12-week horizon)
* Supplier-aware order recommendations
* Inventory simulation with lead-time windows
* Modular internal APIs: `/forecast`, `/seasonality`, `/inventory_projection`, `/`trend etc.
* JSON outputs for easy integration

---

## 🧠 TimeLLM Capabilities

* Input: 24 months of synthetic/real e-commerce data per product (SKU)
* Learns seasonal cycles, price effects, supplier lag, Google Trends, competitor pricing, or marketing events
* Generates forecasts and interpretable attributions (seasonality, trend)

---

## 🧾 Data Schema (partial + synthetic)

| Field                                           | Description             |
| ----------------------------------------------- | ----------------------- |
| `timestamp`                                     | Weekly timestamp        |
| `asin`                                          | Product ID              |
| `title`, `brand`, `department`                  | Metadata                |
| `final_price`, `discount`                       | Price signals           |
| `bs_rank`, `reviews_count`, `bought_past_month` | Demand proxies          |
| `availability`, `is_available`                  | Stock status            |
| `supplier_id`, `lead_time_days`, `max_quantity` | Simulated supplier info |
| `units_sold` (synthetic)                        | Weekly sales            |

---

## 🔄 Data Flow Diagram

```
User/API Call (/ask)
      |
      v
Internal Router
  |        |        |
 /forecast /inventory_projection /logistics
     |             |              |
   TimeLLM      Stock Simulator  Supplier Delay Engine
     |             |              |
     +------> Planner Module <----+
                  |
                  v
               JSON Response
```

---

## 📥 Sample Input

```json
POST /ask
{
  "product_id": "A123",
  "question": "Do I have enough inventory for next 5 weeks? If not, when and how much should I reorder?",
  "current_inventory": 35
}
```

## 📤 Sample Output

```json
{
  "product_id": "A123",
  "weekly_demand_forecast": {
    "2025-W28": 10,
    "2025-W29": 12,
    "2025-W30": 14,
    "2025-W31": 11,
    "2025-W32": 13
  },
  "projected_inventory": {
    "2025-W28": 25,
    "2025-W29": 13,
    "2025-W30": -1,
    "2025-W31": -12,
    "2025-W32": -25
  },
  "recommended_orders": [
    {
      "supplier_id": "Supplier1",
      "order_quantity": 10,
      "order_week": "2025-W27",
      "arrival_week": "2025-W28"
    },
    {
      "supplier_id": "Supplier2",
      "order_quantity": 25,
      "order_week": "2025-W28",
      "arrival_week": "2025-W30"
    }
  ]
}
```

---

## 🧩 Components

* `TimeLLM`: Temporal model with fine-tuned attention over weekly patterns
* `Stock Simulator`: Projects inventory over horizon given current state and orders
* `Supplier Engine`: Models variable delivery windows and constraints
* `Planner`: Optimization layer that decides optimal order plan

---

## 🔐 Limitations

* Depends on synthetic sales and inventory data
* Logistics behavior may not reflect all real-world uncertainties
* Not integrated with real suppliers or ERP/OMS systems

---

## 📅 Roadmap (Stretch Goals)

* Delay likelihood for a supplier is a static number. It should be made product specific and likelihood will decay with time (e.g. delay of 1 day likelihood could be 0.8, 2 days could be 0.65, 3 days could be 0.25 etc.)
* Competitor pricing is being considered by the trends model, which accounts for the same product being sold by multiple suppliers. Which is not accounting for replacement products. For example, a customer may be interested in buying PET water bottles but may notice a deep discount on glass water bottles and choose to buy that instead. 
* Weather forecast impact on logistics
* Increase in tariffs or government policies might impact the supply of one or more products

---

## 📘 License

MIT

---

## 🤝 Contributions

Pull requests welcome! Suggestions, bugs, or extensions? File an issue.

---

## 📎 Related Projects

* [TimeGPT](https://www.nixtla.io/timegpt)
* [NeuralForecast](https://nixtla.github.io/neuralforecast/)
* [Forecasting with Transformers](https://github.com/openai/forecasting)
