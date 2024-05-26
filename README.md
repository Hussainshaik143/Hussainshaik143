import alpaca_trade_api as tradeapi
import time

# Alpaca API credentials
API_KEY = 'your_api_key'
API_SECRET = 'your_api_secret'
BASE_URL = 'https://paper-api.alpaca.markets'  # Use paper trading environment for testing

# Initialize Alpaca API
api = tradeapi.REST(API_KEY, API_SECRET, BASE_URL, api_version='v2')

# Define parameters
symbol = 'AAPL'  # The stock to trade
qty = 10  # The quantity of stock to trade
moving_average_window = 20  # Window for calculating moving average
threshold = 1.02  # Buy threshold, stock price crossing above moving average
stop_loss_threshold = 0.98  # Sell threshold, stock price crossing below moving average

# Function to get moving average of stock price
def get_moving_average(symbol, window):
    bars = api.get_barset(symbol, 'day', limit=window)
    closes = [bar.c for bar in bars[symbol]]
    return sum(closes) / len(closes)

# Main trading logic
def trading_bot():
    while True:
        try:
            # Get current price
            current_price = api.get_last_trade(symbol).price
            
            # Calculate moving average
            moving_average = get_moving_average(symbol, moving_average_window)
            
            # Buy condition: price crosses above moving average
            if current_price > moving_average * threshold:
                api.submit_order(
                    symbol=symbol,
                    qty=qty,
                    side='buy',
                    type='market',
                    time_in_force='gtc'
                )
                print(f"Bought {qty} shares of {symbol} at {current_price}")
            
            # Sell condition: price crosses below moving average
            elif current_price < moving_average * stop_loss_threshold:
                api.submit_order(
                    symbol=symbol,
                    qty=qty,
                    side='sell',
                    type='market',
                    time_in_force='gtc'
                )
                print(f"Sold {qty} shares of {symbol} at {current_price}")
                
            time.sleep(60)  # Check every minute

        except Exception as e:
            print("Error:", str(e))

# Run the trading bot
if __name__ == "__main__":
    trading_bot()
