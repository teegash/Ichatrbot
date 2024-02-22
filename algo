using System;
using System.Linq;
using cAlgo.API;
using cAlgo.API.Indicators;
using cAlgo.API.Internals;
using cAlgo.Indicators;

namespace cAlgo.Robots
{
    [Robot(TimeZone = TimeZones.EasternStandardTime, AccessRights = AccessRights.None)]
    public class _OFX_290320220325_ICHI_RENKO_cBOT : Robot
    {

        [Parameter(DefaultValue = 8, Group = "Ichimoku")]
        public int periodFast { get; set; }
        [Parameter(DefaultValue = 24, Group = "Ichimoku")]
        public int periodMedium { get; set; }
        [Parameter(DefaultValue = 52, Group = "Ichimoku")]
        public int periodSlow { get; set; }

        [Parameter("Instance Name", DefaultValue = "IchiBot")]
        public string InstanceName { get; set; }
        [Parameter("Volume", DefaultValue = 10, Group = "Protection")]
        public int _volume { get; set; }
        [Parameter("Include Break-Even", DefaultValue = true, Group = "Protection")]
        public bool IncludeBreakEven { get; set; }
        [Parameter("Trail after Break-Even", DefaultValue = true, Group = "Protection")]
        public bool Includetrail { get; set; }

        public double chichi;
        public double StopLossInPips, BreakEvenPips, trailingstoppips, TakeProfitInPips, BreakEvenExtraPips;
        public IchimokuKinkoHyo _ichi;
        public AverageTrueRange atr;

        #region onstart
        protected override void OnStart()
        {
            // This is thr initialization logic here            
            atr = Indicators.AverageTrueRange(26, MovingAverageType.Simple);
            _ichi = Indicators.IchimokuKinkoHyo(periodFast, periodMedium, periodSlow);
            InstanceName = InstanceName + "_" + SymbolName;
        }
        #endregion

        #region onBar
        protected override void OnBar()
        {
            TakeProfitInPips = Math.Round((atr.Result.LastValue / Symbol.PipSize), 0);

            //ichimoku
            var kijun = _ichi.KijunSen;
            var tensen = _ichi.TenkanSen;
            var chiku = _ichi.ChikouSpan;
            var sekuA = _ichi.SenkouSpanA;
            var sekuB = _ichi.SenkouSpanB;
//Kumo cloud            
            var cloudupper = Math.Max(sekuA.Last(26), sekuB.Last(26));
            var cloudlower = Math.Min(sekuA.Last(26), sekuB.Last(26));

            var price = Bars.ClosePrices;

//Price is above/below kumo provided that sekuA is above sekuB or otherwise
            var longpricekumo = price.Last(0) > cloudupper && sekuA.Last(0) > sekuB.Last(0);
            var shortpricekumo = price.Last(0) < cloudlower && sekuA.Last(0) < sekuB.Last(0);

//Tensen above Kijun or otherwise            
            var longtensenkijun = tensen.Last(1) > kijun.Last(1);
            var shorttensenkijun = tensen.Last(1) < kijun.Last(1);

//Chiku spann above /below price 26 bars behind            
            var longchikuprice = chiku.Last(1) > Bars.HighPrices.Last(26);
            var shortchikuprice = chiku.Last(1) < Bars.LowPrices.Last(26);

//Kumo falling or rising            
            var kumorising = sekuB.Last(0) > sekuB.Last(1) && sekuA.Last(0) > sekuB.Last(0);
            var kumofalling = sekuB.Last(0) < sekuB.Last(1) && sekuA.Last(0) < sekuB.Last(0);

//Checks if price has crossed above/below the tensen provided that the tensen is above the kijun or otherwise           
            var shortpricetensen = price.HasCrossedBelow(tensen, 1) && tensen.Last(0) < kijun.Last(0);
            var longpricetensen = price.HasCrossedAbove(tensen, 1) && tensen.Last(0) > kijun.Last(0);

//Checks if price has crossed above/below the kijun      
            var longpricekijun = price.Last(1) > kijun.Last(1);
            var shortpricekijun = price.Last(1) < kijun.Last(1);

//Checks if kijun is greater or less than sekouB 26 periods back
            var longkijunsekub = kijun.Last(1) > sekuB.Last(26);
            var shortkijunsekub = kijun.Last(1) < sekuB.Last(26);

//Checks if kijun is falling
            var kijunrising = kijun.Last(0) > kijun.Last(1);
            var kijunfalling = kijun.Last(0) < kijun.Last(1);

//Checks if SekuB is falling or rising            
            var sekubrising = sekuB.Last(0) > sekuB.Last(1);
            var sekubfalling = sekuB.Last(0) < sekuB.Last(1);

//Checks if Bid price equals sekuB 26 periods back    
            var xpricesekub = Symbol.Bid == sekuB.Last(26);

//Checks if the the kumo has flipped
            var longkumoflip = sekuA.Last(0) > sekuB.Last(0);
            var shortkumoflip = sekuA.Last(0) < sekuB.Last(0);

            //Entry Logic Switch
            if (longpricekumo && longchikuprice && longtensenkijun && longkumoflip)
            {
                Open(TradeType.Buy, InstanceName);
            }

            if (shortpricekumo && shortchikuprice && shorttensenkijun && shortkumoflip)
            {
                Open(TradeType.Sell, InstanceName);
            }

            //Exit Logic
            if (longchikuprice)
            {
                Close(TradeType.Sell, InstanceName);
            }

            if (shortchikuprice)
            {
                Close(TradeType.Buy, InstanceName);
            }


        }
        protected override void OnTick()
        {
            if (IncludeBreakEven)
            {
                BreakEvenAdjustment();
            }
        }
        #endregion


        //Function for opening a new trade 
        private void Open(TradeType tradeType, string InstanceName)
        {
//Check there's no existing position before entering a trade
            var position = Positions.Find(InstanceName, SymbolName);
            if (position == null)
            {
                ExecuteMarketOrder(tradeType, SymbolName, _volume / 2, InstanceName, StopLossInPips, TakeProfitInPips);
                ExecuteMarketOrder(tradeType, SymbolName, _volume / 2, InstanceName, StopLossInPips, null);
            }
        }

        #region close trades
//Function for closing trades 
        private void Close(TradeType tradeType, string InstanceName)
        {
            foreach (var position in Positions.FindAll(InstanceName, SymbolName, tradeType))
                ClosePosition(position);
        }
        #endregion

        #region Break Even
// code from clickalgo.com
        private void BreakEvenAdjustment()
        {
           var positn = Positions.Find(InstanceName, SymbolName);
            var allPositions = Positions.FindAll(InstanceName, SymbolName);

            foreach (Position position in allPositions)
            {
                var entryPrice = position.EntryPrice;
                var distance = position.TradeType == TradeType.Buy ? Symbol.Bid - entryPrice : entryPrice - Symbol.Ask;

//Breakeven @ takeprofit                
                BreakEvenPips = TakeProfitInPips;

//Trailing stop is triggered when winning pips equals the 3 times the takeprofit pips                
                trailingstoppips = TakeProfitInPips * 4;
                BreakEvenExtraPips = TakeProfitInPips / 1;

// move stop loss to break even plus and additional (x) pips
                if (distance >= BreakEvenPips * Symbol.PipSize)
                {
                    if (position.TradeType == TradeType.Buy)
                    {
                        if (position.StopLoss <= position.EntryPrice + (Symbol.PipSize * BreakEvenExtraPips) || position.StopLoss == null)
                        {
                            // && position.Pips >= trailingstoppips)
                            if (Includetrail)
                            {
                                //ModifyPosition(position, position.EntryPrice);
                                position.ModifyStopLossPrice(position.EntryPrice + (Symbol.PipSize * BreakEvenExtraPips));
                                Print("Stop Loss to Break Even set for BUY position {0}", position.Id);

                                if (position.Pips >= trailingstoppips)
                                    position.ModifyTrailingStop(true);

                            }
                            else if (!Includetrail)
                            {
                                //ModifyPosition(position, position.EntryPrice + (Symbol.PipSize * BreakEvenExtraPips), position.TakeProfit);
                                position.ModifyStopLossPrice(position.EntryPrice + (Symbol.PipSize * BreakEvenExtraPips));
                                Print("Stop Loss to Break Even set for BUY position {0}", position.Id);
                            }
                        }
                    }
                    else
                    {
                        if (position.StopLoss >= position.EntryPrice - (Symbol.PipSize * BreakEvenExtraPips) || position.StopLoss == null)
                        {
                            // && position.Pips >= trailingstoppips)
                            if (Includetrail)
                            {
                                ModifyPosition(position, entryPrice - (Symbol.PipSize * BreakEvenExtraPips), position.TakeProfit);
                                Print("Stop Loss to Break Even set for SELL position {0}", position.Id);

                                if (position.Pips >= trailingstoppips)
                                    position.ModifyTrailingStop(true);
                            }
                            else if (!Includetrail)
                            {
                                ModifyPosition(position, entryPrice - (Symbol.PipSize * BreakEvenExtraPips), position.TakeProfit);
                                Print("Stop Loss to Break Even set for SELL position {0}", position.Id);

                            }
                        }
                    }

                }
            }
        }

        #endregion
    }
}

