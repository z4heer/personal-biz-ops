The order parameters in the screenshot show:

1. **Transaction & Trigger Type:** `Sell` and `Single` are selected.


2. **Product & Quantity:** `CNC` is selected, and `Qty` is set to `2`.


3. **Trigger Price:** Entered as `1199.9`.


* The purchase average price was **₹1,114.60**.
* Calculating the return at ₹1,199.90:

$$\text{Return} = \frac{1199.90 - 1114.60}{1114.60} = \frac{85.30}{1114.60} \approx +7.65\%$$


* This aligns with the $7+\%$ gain objective.


4. **Order Execution Type:** The order type is currently selected as **Market** instead of **Limit**.


* When `Trigger price` (1199.9) is reached, Zerodha will place a **Market Sell** order into the order book.


* In illiquid ticks or sharp spike-and-fade candles, a market order can experience slippage and execute lower than the trigger level.
* Selecting **Limit** instead allows you to specify a fixed minimum sell price (e.g., Trigger: `1199.00`, Limit Price: `1199.00` or `1198.50`) to avoid slippage.



The configuration will work to capture the $+7.65\%$ target, but it is **partially incorrect in execution choice** due to using a **Market** order instead of a **Limit** order. Switch the radio button to **Limit** to guarantee your minimum exit price.
