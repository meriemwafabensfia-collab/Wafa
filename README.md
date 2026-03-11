import ccxt
import pandas as pd
import pandas_ta as ta
exchange = ccxt.binance()
def
detect_explosion_imminent(symbol):
    # جلب بيانات فريم 15 دقيقة (للدخول المبكر جداً)
  bars = exchange.fetch_ohlcv(symbol, timeframe='15m', limit=100)
      df = pd.DataFrame(bars, columns=['time', 'open', 'high', 'low', 'close', 'volume'])
      # 1. مؤشر البولنجر باند لقياس "الضغط"
          bbands = ta.bbands(df['close'], length=20, std=2)
              df = pd.concat([df, bbands], axis=1)

                      # حساب عرض النطاق (Bandwidth) - كلما قل، زاد احتمال الانفجار
                          df['bw'] = (df['BBU_20_2.0'] - df['BBL_20_2.0']) / df['BBM_20_2.0']

                                  # 2. مؤشر تدفق الأموال (MFI) لقياس دخول السيولة قبل السعر
                                      df['mfi'] = ta.mfi(df['high'], df['low'], df['close'], df['volume'], length=14)

                                              # 3. الاستوكاستيك (المؤشر المفضل لديك في الصور)
                                                  stoch = ta.stochrsi(df['close'], length=14, rsi_length=14, k=3, d=3)
                                                      df = pd.concat([df, stoch], axis=1)

                                                          # القيم الحالية
                                                              current_bw = df['bw'].iloc[-1]
                                                                  current_mfi = df['mfi'].iloc[-1]
                                                                      current_k = df.iloc[-1]['STOCHRSIk_14_14_3_3']
                                                                          last_vol = df['volume'].iloc[-1]
                                                                              avg_vol = df['volume'].tail(10).mean()

                                                                                  print(f"🔍 فحص دقيق لـ {symbol}...")

                                                                                      # --- شروط "قبيل الانفجار" ---
                                                                                          # شرط 1: انضغاط السعر (Bandwidth منخفض جداً)
                                                                                              is_squeezed = current_bw < 0.02 
                                                                                                  # شرط 2: دخول سيولة خفية (MFI يرتفع والسعر لسه مستقر)
                                                                                                      is_money_flowing = current_mfi > 60
                                                                                                          # شرط 3: بداية تقاطع إيجابي في الاستوكاستيك من القاع (تحت 20)
                                                                                                              is_stoch_rising = current_k < 30 

                                                                                                                  if is_squeezed and is_money_flowing:
                                                                                                                          print(f"🌟 [WAFA ALERT]: انفجار وشيك على {symbol}!")
                                                                                                                                  print(f"السبب: ضغط سعري عالي + دخول سيولة ذكية.")
                                                                                                                                      elif last_vol > (avg_vol * 3):
                                                                                                                                              print(f"🔥 [WAFA ALERT]: الانفجار بدأ الآن على {symbol}! سيولة ضخمة دخلت.")

                                                                                                                                              # قائمة المراقبة
                                                                                                                                              symbols = ['ARC/USDT', 'XPIN/USDT', 'SOL/USDT']
                                                                                                                                              for s in symbols:
                                                                                                                                                  try: detect_explosion_imminent(s)
                                                                                                                                                      except: continue     