# task1
solution

use std::collections::VecDeque;
pub struct RollingOHLC 
{
    window_duration: u64, 
    prices: VecDeque<(u64, f64)>, 
}

impl RollingOHLC 
{
    pub fn new(window_duration: u64) -> Self 
    {
        RollingOHLC 
        {
            window_duration,
            prices: VecDeque::new(),
        }
    }
    pub fn update(&mut self, timestamp: u64, price: f64) 
    {
        let expiration_time = timestamp - self.window_duration;
        while let Some((first_timestamp, _)) = self.prices.front() 
        {
            if *first_timestamp <= expiration_time 
            {
                self.prices.pop_front();
            } else 
            {
                break;
            }
        }
        self.prices.push_back((timestamp, price));
    }
    pub fn calculate_ohlc(&self) -> Option<(f64, f64, f64, f64)> 
    {
        if self.prices.is_empty()
        {
            return None;
        }
        let (first_timestamp, _) = *self.prices.front().unwrap();
        let (_, last_price) = *self.prices.back().unwrap();
        let mut min_price = last_price;
        let mut max_price = last_price;
        let mut open_price = last_price;
        for (_, price) in &self.prices
        {
            if *price < min_price
            {
                min_price = *price;
            }
            if *price > max_price
            {
                max_price = *price;
            }
        }

        Some((open_price, max_price, min_price, last_price))
    }
}

#[cfg(test)]
mod tests 
{
    use super::*;
    #[test]
    fn test_rolling_ohlc() 
    {
        let mut rolling_ohlc = RollingOHLC::new(300); 
        rolling_ohlc.update(100, 50.0);
        rolling_ohlc.update(200, 60.0);
        rolling_ohlc.update(300, 55.0);
        rolling_ohlc.update(400, 70.0);
        rolling_ohlc.update(500, 65.0);
        let ohlc = rolling_ohlc.calculate_ohlc().unwrap();
        assert_eq!(ohlc, (50.0, 70.0, 50.0, 65.0));
    }
}
