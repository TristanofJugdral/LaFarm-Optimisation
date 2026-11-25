# MATH301 - LaFarm Optimisation Model

# Set OBJECTIVE = ... ("nutrition", "poundage") **lowercase**
OBJECTIVE = "nutrition"

# Set demand percent (**NOTE: this value doesn’t affect the .html file)
DEMAND_FRACTION = 0.50

# !pip install gurobipy
import gurobipy as gp # Optimisation
import pandas as pd
from pathlib import Path
from gurobipy import GRB, multidict
import math
import gurobipy as gp
from gurobipy import GRB, multidict

# CROP DATA
# ============================================================
# Each crop contains:
# [Poundage        [lbs per bed-foot],
#  Maturity Time   [weeks],
#  Nutrition Score        ,
#  Historical Distribution]
# ============================================================
crops, poundage, maturity, nutrition, distribution = multidict({
   "Beets":                    [2.00,  7,  0.40,  611.00],
   "Broccoli":                 [1.00,  8,  1.23,  182.66],
   "Cabbage":                  [4.00, 12,  0.47,  854.14],
   "Carrot":                   [3.00, 11,  0.72, 1133.27],
   "Cucumber":                 [2.50,  6,  0.75, 1286.71],
   "Eggplant":                 [3.00,  9,  0.15,  853.50],
   "Garlic":                   [0.75, 41,  0.01,  270.53],  # Provisional: week 40 to next year week 28
   "Kale":                     [2.00,  7,  0.40,  263.55],
   "Kohlrabi":                 [0.70,  9,  1.22,   12.50],
   "Leeks":                    [1.50, 11,  0.54,  254.80],
   "Lettuce (Romaine)":        [2.00,  7,  0.37,  387.27],
   "Onion":                    [2.00, 15,  0.21,  305.59],
   "Pak Choi":                 [3.00,  4,  0.53,  677.32],
   "Peas":                     [0.50,  9,  1.38,   97.37],
   "Peppers (Bell)":           [1.50,  9,  2.53,  513.41],
   "Peppers (Hot)":            [1.00,  9,  0.80,  130.22],
   "Potatoes":                 [2.00, 16,  1.71,  704.50],
   "Radish":                   [1.25,  4,  0.37,  362.91],
   "Scallions":                [1.00,  6,  0.10,   80.51],
   "Squash (Summer)":          [2.50,  5,  1.00, 2241.63],
   "Squash (Winter)":          [1.50, 12,  0.62,  534.43],
   "Sweet Potatoes":           [2.00, 13,  1.83,  443.16],
   "Swiss Chard":              [1.00,  7,  0.33,  444.01],
   "Tomatoes (Cherry)":        [2.00,  9,  1.13, 1361.86],
   "Tomatoes (Slicing/Paste)": [4.00, 10,  0.87, 3942.47],
   "Turnips":                  [1.50,  5,  0.46,  424.88]
})

# WATER USAGE DATA (GALLONS PER BED-FOOT)
# ============================================================
water = {
   "Beets": 66.38,
   "Broccoli": 44.86,
   "Cabbage": 207.36,
   "Carrot": 133.85,
   "Cucumber": 41.44,
   "Eggplant": 182.54,
   "Garlic": 2041.16,
   "Kale": 345.59,
   "Kohlrabi": 18.82,
   "Leeks": 45.87,
   "Lettuce (Romaine)": 176.40,
   "Onion": 65.98,
   "Pak Choi": 155.52,
   "Peas": 28.15,
   "Peppers (Bell)": 62.89,
   "Peppers (Hot)": 110.88,
   "Potatoes": 38.33,
   "Radish": 29.33,
   "Scallions": 181.44,
   "Squash (Summer)": 57.86,
   "Squash (Winter)": 58.65,
   "Sweet Potatoes": 83.74,
   "Swiss Chard": 75.60,
   "Tomatoes (Cherry)": 44.25,
   "Tomatoes (Slicing/Paste)": 88.51,
   "Turnips": 33.46
}

# DOWNSTREAM IMPACTS DATA
# ============================================================
# [Water Efficiency, Wholesale Price ($/lb), Labour Hours (per 100-foot bed)]
downstream_crops, water_efficiency, wholesale_price, labor = multidict({
   "Beets":                    [0.06711409396, 2.00, 14.9],
   "Broccoli":                 [0.06024096386, 2.00, 8.3],
   "Cabbage":                  [0.1361933946,  1.10, 26.7],
   "Carrot":                   [0.07575757576, 1.65, 24.0],
   "Cucumber":                 [0.1011940903,  1.35, 18.3],
   "Eggplant":                 [0.05599104143, 1.90, 28.2],
   "Garlic":                   [0.01293103448, 5.00, 11.6],
   "Kale":                     [0.1011122346,  2.30, 8.6],
   "Kohlrabi":                 [0.03888888889, 1.50, 12.0],
   "Leeks":                    [0.1095290252,  1.65, 8.3],
   "Lettuce (Romaine)":        [0.04556846662, 3.30, 13.3],
   "Onion":                    [0.1334668001,  1.35, 11.1],
   "Pak Choi":                 [0.08720930233, 4.00, 8.6],
   "Peas":                     [0.04464285714, 4.00, 2.8],
   "Peppers (Bell)":           [0.03191489362, 5.00, 9.4],
   "Peppers (Hot)":            [0.04255319149, 2.50, 9.4],
   "Potatoes":                 [0.08592910849, 1.75, 13.3],
   "Radish":                   [0.06578947368, 1.90, 10.0],
   "Scallions":                [0.01365001365, 6.60, 11.1],
   "Squash (Summer)":          [0.1610305958,  1.35, 11.5],
   "Squash (Winter)":          [0.1185770751,  1.10, 11.5],
   "Sweet Potatoes":           [0.1367053999,  1.10, 13.3],
   "Swiss Chard":              [0.1402524544,  2.30, 3.1],
   "Tomatoes (Cherry)":        [0.1125175809,  2.25, 7.9],
   "Tomatoes (Slicing/Paste)": [0.1012658228,  2.50, 15.8],
   "Turnips":                  [0.06578947368, 1.90, 12.0]
})

# PLANTING WINDOWS
# ============================================================
# [Earliest possible planting week, Latest possible planting week]
# ============================================================
window_crops, first_week, last_week = multidict({
   "Beets":                    [13, 19],
   "Broccoli":                 [15, 34],
   "Cabbage":                  [14, 18],
   "Carrot":                   [13, 33],
   "Cucumber":                 [18, 33],
   "Eggplant":                 [19, 33],
   "Garlic":                   [40, 40],
   "Kale":                     [15, 19],
   "Kohlrabi":                 [14, 19],
   "Leeks":                    [14, 28],
   "Lettuce (Romaine)":        [13, 19],
   "Onion":                    [13, 29],
   "Pak Choi":                 [19, 23],
   "Peas":                     [13, 19],
   "Peppers (Bell)":           [18, 31],
   "Peppers (Hot)":            [18, 31],
   "Potatoes":                 [13, 33],
   "Radish":                   [13, 35],
   "Scallions":                [13, 34],
   "Squash (Summer)":          [18, 34],
   "Squash (Winter)":          [18, 27],
   "Sweet Potatoes":           [20, 28],
   "Swiss Chard":              [14, 19],
   "Tomatoes (Cherry)":        [19, 30],
   "Tomatoes (Slicing/Paste)": [19, 30],
   "Turnips":                  [13, 39]
})

# MODEL PARAMETERS
# ============================================================

# Maximum bed-feet that can be occupied at any one time
TOTAL_BEDFEET = 7382

# 52 one-week periods
weeks = range(1, 53)
YEAR_WEEKS = 52
# These crops may occupy land across the boundary of the repeating annual plan.
CYCLIC_CROPS = {"Garlic"}

maturity_weeks = {
   c: math.ceil(maturity[c])
   for c in crops
}
if any(not 1 <= maturity_weeks[c] <= YEAR_WEEKS for c in crops):
    raise ValueError("Crop occupancy durations must be between 1 and 52 weeks.")

def occupies_week(crop, planting_week, week):
    """Occupancy of a crop cohort in a repeating 52-week annual plan."""
    return (week - planting_week) % YEAR_WEEKS < maturity_weeks[crop]

# DISTRIBUTION REQUIREMENTS
minimum_distribution = {
   c: DEMAND_FRACTION * distribution[c]
   for c in crops
}

# POSSIBLE START WEEKS
start_weeks = {}
for c in crops:
   start_weeks[c] = [
       w
       for w in range(
           first_week[c],
           last_week[c] + 1
       )
       if c in CYCLIC_CROPS or w + maturity_weeks[c] - 1 <= YEAR_WEEKS
   ]

# Create all valid combinations of crop and planting week
crop_week_combinations = gp.tuplelist(
   (c, w)
   for c in crops
   for w in start_weeks[c]
)

# CREATE MODEL (Gurobi)
model = gp.Model("LaFarm")

# DECISION VARIABLES
# ============================================================
# bedfeet[c,w]: # of bed-feet of crop c planted beginning in week w.

bedfeet = model.addVars(
   crop_week_combinations,
   vtype=GRB.CONTINUOUS,
   lb=0,
   name="BedFeet"
)

# start[c,w]
# 1 = crop begins in week w
# 0 = it does not
start = model.addVars(crop_week_combinations, vtype=GRB.BINARY, name="Start")

# use_crop[c]
# 1 = crop appears in planting plan
# 0 = crop is not planted
use_crop = model.addVars(crops, vtype=GRB.BINARY, name="UseCrop")

# NUTRITION OBJECTIVE
# ============================================================
if OBJECTIVE == "nutrition":
   model.setObjective(
       gp.quicksum(
           nutrition[c] * bedfeet[c, w]
           for c, w in crop_week_combinations
       ),
       GRB.MAXIMIZE
   )

# POUNDAGE OBJECTIVE
# ============================================================
elif OBJECTIVE == "poundage":
   model.setObjective(
       gp.quicksum(
           poundage[c] * bedfeet[c, w]
           for c, w in crop_week_combinations
       ),
       GRB.MAXIMIZE
   )

## CONSTRAINTS:
MIN_BEDFEET_IF_USED = 1
MIN_DIVERSITY = 1

# CONSTRAINT 1: EACH CROP CAN ONLY BE PLANTED ONCE
# ** If crop is used --> exactly one start week.
# ============================================================
model.addConstrs(
   (
       gp.quicksum(
           start[c, w]
           for w in start_weeks[c]
       )
       == use_crop[c]
       for c in crops
   ),
   name="PlantOnce"
)

# CONSTRAINT 2: LINK BED-FEET TO SELECTED START WEEK
# ============================================================
model.addConstrs(
   (bedfeet[c, w] <= TOTAL_BEDFEET * start[c, w]
       for c, w in crop_week_combinations
   ),
   name="StartLinkUpper"
)

model.addConstrs(
   (bedfeet[c, w] >= MIN_BEDFEET_IF_USED * start[c, w]
       for c, w in crop_week_combinations
   ),
   name="StartLinkLower"
)

# CONSTRAINT 3: WEEKLY BED-SPACE LIMIT
# Total land occupied in EVERY week must be <= 7382 feet
# ============================================================
model.addConstrs(
   (
       gp.quicksum(
           bedfeet[c, start_w]
           for c in crops
           for start_w in start_weeks[c]
           if occupies_week(c, start_w, week)
       )
       <= TOTAL_BEDFEET
       for week in weeks
   ),
   name="WeeklyBedSpace"
)

# CONSTRAINT 4: DISTRIBUTION / DEMAND CONSTRAINT
# Production must be at least 50% of historical distribution 
# (according to DEMAND_FRACTION = 50%)
# ============================================================
demand_constraints = model.addConstrs(
   (
       poundage[c]
       * gp.quicksum(
           bedfeet[c, w]
           for w in start_weeks[c]
       )
       >= minimum_distribution[c]
       for c in crops
   ),
   name="DistributionRequirement"
)


# OPTIMIZE MODEL
# INTERACTIVE CALENDAR AND DEMAND SENSITIVITY (NO CSV)
# ============================================================
# Settings (edit these and rerun to regenerate the offline dashboard)
DEMAND_INCREMENT_PERCENT = 1
MAX_DEMAND_PERCENT = 110       # Allows us to go to 110% demand (over the requirement)
SENSITIVITY_BIN_PERCENT = 5    # 77% uses 75–80%; 80% uses 80–85%.
OBJECTIVES = ("nutrition", "poundage")
CALENDAR_YEAR = 2026
SOLVER_GAP = 0.0001
OPEN_BROWSER = True
CRITICAL_CANDIDATE_LIMIT = 5

import json
import sys
import webbrowser
from datetime import date
from pathlib import Path
critical_only = '--critical-only' in sys.argv

if DEMAND_INCREMENT_PERCENT != 1:
    raise ValueError('This slider uses 1% increments; keep DEMAND_INCREMENT_PERCENT = 1.')
if MAX_DEMAND_PERCENT < 50 or MAX_DEMAND_PERCENT > 200:
    raise ValueError('Choose a maximum demand between 50 and 200%.')
if MAX_DEMAND_PERCENT % SENSITIVITY_BIN_PERCENT:
    raise ValueError('Maximum demand must be a multiple of the sensitivity bin width.')

# Colours match their respective crops (Note: uses American spelling)
crop_colors = ['#9C3159','#357342','#80A65B','#DF862C','#409477',
               '#70458C','#C5B88A','#355C39','#97A550','#6D9858',
               '#8BB950','#B79849','#447E45','#65A34B','#D4A72D',
               '#BF4337','#A98554','#CE4E78','#63884B','#DFA92A',
               '#C48438','#C26F35','#B64556','#CF503D','#BA3F32','#94649B']
if not critical_only and len(crops) != len(crop_colors):
    raise ValueError('Add a color for every crop.')

runs = {}
for objective in (("nutrition",) if critical_only else OBJECTIVES):
    coefficients = nutrition if objective == "nutrition" else poundage
    model.setObjective(
        gp.quicksum(coefficients[c] * bedfeet[c, w]
                    for c, w in crop_week_combinations),
        GRB.MAXIMIZE
    )
    scenarios = []
    model.Params.OutputFlag = 0
    model.Params.MIPGap = SOLVER_GAP 
    model.Params.IntegralityFocus = 1 # Added this line to fix eggplant issue when finding poundage results


    for demand_percent in range(0, MAX_DEMAND_PERCENT + 1, DEMAND_INCREMENT_PERCENT):
        demand_fraction = demand_percent / 100
        for c in crops:
            demand_constraints[c].RHS = demand_fraction * distribution[c]
        print(f'{objective.capitalize()}: solving {demand_percent}% demand...', flush=True)
        model.optimize()
        scenario = dict(percent=demand_percent, status=int(model.Status),
                        ok=model.Status == GRB.OPTIMAL)
        scenarios.append(scenario)
        if not scenario['ok']:
            print(f'No optimal result at {demand_percent}%; status {model.Status}.')
            continue
        scenario['gap'] = model.MIPGap
        rows = []
        for c in crops:
            allocated = sum(bedfeet[c, w].X for w in start_weeks[c])
            selected = next((w for w in start_weeks[c] if start[c, w].X > .5), None)
            selected = selected if allocated > 0.0001 else None
            pounds = poundage[c] * allocated
            rows.append(dict(crop=c, start=selected,
                             end=(selected+maturity_weeks[c]-2) % YEAR_WEEKS + 1 if selected is not None else None,
                             endUnwrapped=selected+maturity_weeks[c]-1 if selected is not None else None,
                             harvestCycle=(selected+maturity_weeks[c]-2) // YEAR_WEEKS if selected is not None else None,
                             duration=maturity_weeks[c], bed=allocated,
                             demand=demand_fraction*distribution[c], pounds=pounds,
                             surplus=pounds-demand_fraction*distribution[c],
                             nutrition=nutrition[c]*allocated, water=water[c]*allocated,
                             revenue=wholesale_price[c]*pounds, labor=labor[c]*allocated/100,
                             efficiency=water_efficiency[c]))
        scenario['objectiveValue'] = model.ObjVal
        scenario['rows'] = rows
        scenario['weekly'] = [sum(r['bed'] for r in rows if r['start'] is not None
                                  and (w-r['start']) % YEAR_WEEKS < r['duration']) for w in weeks]
        scenario['total'] = {key:sum(r[key] for r in rows)
                             for key in ['bed','pounds','nutrition','water','revenue','labor']}
        if max(scenario['weekly']) > TOTAL_BEDFEET + 1e-3:
            raise RuntimeError('Weekly land constraint verification failed.')
        if any(r['surplus'] < -1e-3 for r in rows):
            raise RuntimeError('Demand constraint verification failed.')

    valid = [s for s in scenarios if s['ok']]
    if not valid:
        raise RuntimeError(f'No optimal scenarios for {objective}.')
    runs[objective] = scenarios

def find_critical_nutrition_demand(scenarios):
    """Refine the largest observed allocation shifts between distinct schedules."""
    candidates = []
    for before, after in zip(scenarios, scenarios[1:]):
        if not before['ok'] or not after['ok']:
            continue
        if all(a['start'] == b['start'] for a, b in zip(before['rows'], after['rows'])):
            continue
        shift = sum(abs(a['bed'] - b['bed']) for a, b in zip(before['rows'], after['rows']))
        candidates.append((shift, before, after))
    candidates.sort(key=lambda candidate: candidate[0], reverse=True)

    model.update()
    with model.copy() as full:
        full.Params.MIPGap = 1e-8
        full.setObjective(gp.quicksum(
            nutrition[c] * full.getVarByName(var.VarName)
            for (c, week), var in bedfeet.items()
        ), GRB.MAXIMIZE)
        full.update()

        def solve_at(problem, percent):
            for crop in crops:
                constraint = problem.getConstrByName(demand_constraints[crop].ConstrName)
                constraint.RHS = distribution[crop] * percent / 100
            problem.optimize()
            return problem.ObjVal if problem.Status == GRB.OPTIMAL else None

        def fix_schedule(problem, scenario):
            chosen = {(row['crop'], row['start']) for row in scenario['rows']}
            for key, var in start.items():
                fixed = problem.getVarByName(var.VarName)
                fixed.LB = fixed.UB = int(key in chosen)

        score_tolerance = 0.001
        for shift, before, after in candidates[:CRITICAL_CANDIDATE_LIMIT]:
            low, high = before['percent'], after['percent']
            with full.relax() as plan_a, full.relax() as plan_b:
                fix_schedule(plan_a, before)
                fix_schedule(plan_b, after)

                def difference(percent):
                    a = solve_at(plan_a, percent)
                    b = solve_at(plan_b, percent)
                    return None if a is None or b is None else a - b

                left, right = difference(low), difference(high)
                if left is None or right is None or left <= score_tolerance or right >= -score_tolerance:
                    continue
                while high - low > 0.00001:
                    mid = (low + high) / 2
                    gap = difference(mid)
                    if gap is None:
                        break
                    if gap > 0:
                        low = mid
                    else:
                        high = mid
                if high - low > 0.00001:
                    continue

                crossover = (low + high) / 2
                verified = True
                for offset, preferred in [(-0.001, plan_a), (0.001, plan_b)]:
                    percent = crossover + offset
                    if not 0 <= percent <= MAX_DEMAND_PERCENT:
                        verified = False
                        break
                    expected = solve_at(preferred, percent)
                    other = solve_at(plan_b if preferred is plan_a else plan_a, percent)
                    actual = solve_at(full, percent)
                    if (expected is None or other is None or actual is None
                            or expected <= other + score_tolerance
                            or full.ObjBound > expected + score_tolerance):
                        verified = False
                        break
                if verified:
                    return dict(percent=crossover, lower=before['percent'],
                                upper=after['percent'], allocationShift=shift)
    return None


critical = find_critical_nutrition_demand(runs['nutrition'])
if critical is None:
    critical_message = (
        f"No nutrition schedule crossover verified among the "
        f"{CRITICAL_CANDIDATE_LIMIT} largest candidate intervals."
    )
else:
    critical_message = (
        f"Critical nutrition demand: {critical['percent']:.2f}% "
        f"(slider interval {critical['lower']}–{critical['upper']}%)."
    )
print(critical_message, flush=True)
if critical_only:
    raise SystemExit(0)

try:
    from plotly.offline import get_plotlyjs
except ImportError:
    import sys
    raise SystemExit(
        'Install Plotly in Terminal, then rerun:\n'
        f'"{sys.executable}" -m pip install plotly'
    )

# The seasons are based off of Easton's seasons (i.e, ~21 December starts Winter)
week_one_start = date(CALENDAR_YEAR, 1, 1)
def week_position(month, day):
    return .5 + (date(CALENDAR_YEAR, month, day)-week_one_start).days/7

spring = week_position(3, 1)
summer = week_position(6, 1)
fall = week_position(9, 1)
winter = week_position(12, 1)
seasons = [dict(name='Winter', start=.5, end=spring),
           dict(name='Spring', start=spring, end=summer),
           dict(name='Summer', start=summer, end=fall),
           dict(name='Fall', start=fall, end=winter),
           dict(name='Winter', start=winter, end=52.5)]

all_valid = [s for scenarios in runs.values() for s in scenarios if s['ok']]
payload = dict(criticalMessage=critical_message, defaultObjective=OBJECTIVE if OBJECTIVE in OBJECTIVES else 'nutrition',
               objectives=list(OBJECTIVES), crops=list(crops), colors=crop_colors,
               binWidth=SENSITIVITY_BIN_PERCENT, maxDemand=MAX_DEMAND_PERCENT,
               year=CALENDAR_YEAR, weekOneStart=week_one_start.isoformat(),
               capacity=TOTAL_BEDFEET, solverGap=SOLVER_GAP, seasons=seasons,
               cyclicCrops=sorted(CYCLIC_CROPS), yearWeeks=YEAR_WEEKS,
               garlicDuration=maturity_weeks['Garlic'],
               runs=runs,
               scales={key:max(1,max(r[key] for s in all_valid for r in s['rows']))*1.12
                       for key in ['bed','pounds','nutrition','water','revenue','labor']})

# Both datasets and the plotting library are embedded here so that we can use it offline

HTML_TEMPLATE = r"""<!doctype html>
<html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1"><title>LaFarm | Demand sensitivity</title>
<style>
*{box-sizing:border-box}body{margin:0;background:#f3f5ef;color:#213c2b;font:16px/1.5 system-ui,-apple-system,sans-serif}header{background:#213e2e;color:#fff;padding:28px max(24px,calc((100vw - 1440px)/2))}header h1{font-size:32px;line-height:1.2;margin:0 0 8px}header p{margin:0;color:#d9e6d7}main{max-width:1488px;margin:auto;padding:0 24px 40px}.controls{position:sticky;top:0;z-index:20;background:#f3f5eff8;backdrop-filter:blur(8px);padding:18px 0;display:grid;grid-template-columns:minmax(300px,2fr) minmax(160px,1fr) minmax(210px,1fr) auto;gap:28px;align-items:end;border-bottom:1px solid #d4ddce}.control label{font-weight:650;display:block;margin-bottom:8px}.slider{width:100%;accent-color:#3e7349;display:block;margin:10px 0}.slider-label{display:flex;justify-content:space-between;gap:16px}.slider-label output{font-size:22px;font-weight:750}.ticks{display:flex;justify-content:space-between;font-size:12px;color:#526752}select,button{font:inherit;padding:9px 12px;border:1px solid #a8b9a4;border-radius:6px;background:#fff;color:#213c2b}select{width:100%}button{cursor:pointer}#window{font-size:13px;font-weight:400;color:#536551}.status{padding:12px 0;font-size:14px}.cards{display:grid;grid-template-columns:repeat(5,1fr);gap:12px;margin:8px 0 24px}.card{background:#fff;border:1px solid #dbe3d5;border-radius:9px;padding:15px}.card .value{font-size:24px;font-weight:720;line-height:1.4}.card .range{font-size:12px;color:#586b56;margin-top:5px}section{background:#fff;border:1px solid #dbe3d5;border-radius:12px;padding:24px;margin:20px 0}h2{font-size:23px;line-height:1.3;margin:0 0 8px}.subtitle{color:#566953;margin:0 0 12px}.plot-scroll{overflow-x:auto}.plot{min-width:940px;width:100%}figure{margin:0}.caption{font-size:14px;line-height:1.6;color:#4e624e;padding-top:14px;margin:0}.key{display:flex;gap:20px;flex-wrap:wrap;align-items:center;font-size:13px;color:#4e624e;margin:12px 0}.solid{display:inline-block;width:28px;height:12px;background:#597d43;vertical-align:middle;margin-right:6px}.soft{display:inline-block;width:28px;height:12px;background:#597d4380;filter:blur(2px);vertical-align:middle;margin-right:6px}.note{background:#edf2e7;border-left:3px solid #7e9a69;padding:13px 17px;margin:14px 0}.table-scroll{overflow-x:auto}table{font-size:13px;border-collapse:collapse;min-width:1400px;width:100%}th,td{text-align:right;padding:10px;border-bottom:1px solid #e2e9db;white-space:nowrap}th{background:#edf2e7}th:first-child,td:first-child{text-align:left}details summary{cursor:pointer;font-weight:600}.error{color:#a83232}@media(min-width:851px) and (max-width:1150px){.controls{grid-template-columns:1fr 1fr;gap:14px}}@media(max-width:850px){.controls{grid-template-columns:1fr;gap:12px;position:static}.cards{grid-template-columns:repeat(2,1fr)}section{padding:16px}main{padding:0 14px 24px}}@media print{.controls{position:static}button,.modebar{display:none!important}section{break-inside:avoid}body{background:#fff}}
</style></head><body>
<header><h1>LaFarm · Demand sensitivity</h1><p id="priority"></p></header>
<main><div class="controls"><div class="control"><div class="slider-label"><label for="demand">% demand</label><output id="demandValue" for="demand">50%</output></div><input class="slider" id="demand" type="range" min="0" step="1" value="50" aria-label="Demand percentage"><div class="ticks"><span>0%</span><span id="window"></span><span id="maximum"></span></div></div><div class="control"><label for="objective">Optimization priority</label><select id="objective"><option value="nutrition">Nutrition</option><option value="poundage">Poundage</option></select></div><div class="control"><label for="metric">Crop impact metric</label><select id="metric"></select></div><button id="print">Print / save PDF</button></div>
<p id="critical-demand" class="note"></p><div id="status" class="status" role="status"></div><div class="cards" id="cards"></div>
<section><h2>Seasonal planting calendar · Easton, Pennsylvania</h2><p class="subtitle" id="seasonDefinition"></p><figure><div class="plot-scroll"><div id="calendar" class="plot"></div></div><figcaption class="caption">Solid bars show the selected plan. The soft, faint bars show planting intervals in the surrounding demand scenarios. Every faint interval is a separate plan, rather than an extension of the selected harvest. Thin lines segment the calendar into season boundaries. A crossing-year crop is split into two matching segments: the early segment is carry-in from last year; the late segment is this year’s planting continuing into next year.</figcaption></figure><div class="key"><span><i class="solid"></i>Selected demand</span><span><i class="soft"></i>Neighboring demand plans</span></div></section>
<section><h2>Crop allocation and downstream impact</h2><p class="subtitle" id="metricSubtitle"></p><figure><div class="plot-scroll"><div id="impact" class="plot"></div></div><figcaption class="caption" id="impactCaption"></figcaption></figure><div class="key"><span><i class="solid"></i>Selected-demand result</span><span><i class="soft"></i>Minimum–maximum within the enclosing 5-point interval</span><span>Thin marks identify the exact range endpoints.</span></div></section>
<section><h2>Weekly use of farm space</h2><figure><div class="plot-scroll"><div id="capacity" class="plot"></div></div><figcaption class="caption">The solid line is occupied bed-feet in the selected plan. The soft envelope shows the minimum and maximum weekly occupancy among nearby demand scenarios. The dashed line is farm capacity. Unused space is capacity minus occupancy for this plan.</figcaption></figure></section>
<section><h2>What changes as demand becomes stricter?</h2><figure><div class="plot-scroll"><div id="trend" class="plot"></div></div><figcaption class="caption">The line compares total impact under the same optimization priority at every demand level. The shaded vertical window marks the neighboring scenarios; the dot marks the slider selection. Click a point to change the slider.</figcaption></figure></section>
<section><h2>Interpretation and crop data</h2><p class="note">Sensitivity is grouped into <b>5-percentage-point intervals</b>: selecting 77% compares every integer demand from 75% through 80%. Selecting 80% starts the next interval, 80–85%. At the maximum slider value, the final interval is used. The slider itself still moves in 1% steps. Soft bands show modeled sensitivity to that input; they are not confidence intervals, probabilities, or solver tolerances. Each scenario is solved separately. A different optimal schedule can be chosen even when several schedules perform equally well.</p><p>This view replaces the earlier 1% objective-loss whiskers. It compares demand scenarios directly and permits planting dates and crop allocations to change. Each crop’s minimum and maximum may occur in different plans, so the displayed extrema cannot all be combined into a single feasible plan.</p><p>Nutrition is a supplied score per bed-foot, unique to each crop. Revenue is estimated using a crop’s wholesale value, not profit. Water and labor are modeled resource estimates. The Optimization priority dropdown switches between independently optimized nutrition and poundage plans. Both are precomputed, so switching the priority still works offline without rerunning Python. The Crop impact metric dropdown changes only what is plotted. All models use the same constraints. Axis scales are shared between priorities for comparison.</p><p class="note">This is a repeating annual plan, not a farm starting with empty beds on January 1. For instance, garlic may continue across week 52, and the previous year’s equivalent planting occupies the early-year beds. Both segments count against weekly land capacity. Production, water, labor and revenue are counted once per annual crop cycle, not once per displayed segment. Garlic uses a provisional 41-week occupancy: week 40 through week 28 of the following year, consistent with fall planting and a mid-July harvest. Other crop-duration inputs are unchanged.</p><p id="solverNote"></p><details><summary>Show the selected plan’s downstream impacts</summary><div class="table-scroll" id="table"></div></details><p class="caption">Season definition: <a href="https://www.noaa.gov/education/multimedia/infographic/infographic-meteorological-and-astronomical-seasons" target="_blank" rel="noopener">NOAA meteorological seasons</a>. Seasons provide calendar context; they do not add frost or weather constraints. Garlic timing reference: <a href="https://extension.psu.edu/garlic-production" target="_blank" rel="noopener">Penn State Extension garlic production</a> (fall planting, harvest generally by mid-July).</p></section>
</main><script>__PLOTLY_LIBRARY__</script>
<script>
const PACKAGE=__PAYLOAD__;
document.getElementById("critical-demand").textContent = PACKAGE.criticalMessage;
let DATA={...PACKAGE,objective:PACKAGE.defaultObjective,scenarios:PACKAGE.runs[PACKAGE.defaultObjective]};
const el=id=>document.getElementById(id);
const metrics={bed:['Bed-feet allocated','bed-feet'],pounds:['Expected production','lb'],nutrition:['Nutrition contribution','score'],water:['Estimated water use','gallons'],revenue:['Estimated wholesale value','USD'],labor:['Estimated labor','hours']};
const format=(v,d=1)=>v===null||v===undefined?'—':Number(v).toLocaleString(undefined,{maximumFractionDigits:d});
const escapeHtml=s=>String(s).replaceAll('&','&amp;').replaceAll('<','&lt;').replaceAll('>','&gt;');
const config={responsive:true,displaylogo:false,scrollZoom:false,toImageButtonOptions:{format:'svg',filename:'lafarm-chart'}};
const common={paper_bgcolor:'rgba(0,0,0,0)',plot_bgcolor:'#fff',font:{family:'Arial, sans-serif',size:13,color:'#27402e'},hovermode:'closest',showlegend:false,margin:{l:190,r:35,t:65,b:65},transition:{duration:0}};
function layout(extra={}){return {...common,...extra};}
function axisY(){return {categoryorder:'array',categoryarray:DATA.crops,autorange:'reversed',automargin:true};}
function rangeWindow(p){let low=Math.floor(p/DATA.binWidth)*DATA.binWidth;if(low===DATA.maxDemand)low=Math.max(0,low-DATA.binWidth);const high=Math.min(DATA.maxDemand,low+DATA.binWidth);return {low,high,all:DATA.scenarios.filter(s=>s.percent>=low&&s.percent<=high),valid:DATA.scenarios.filter(s=>s.percent>=low&&s.percent<=high&&s.ok)};}
function extrema(scenarios,key){return DATA.crops.map((c,i)=>{const a=scenarios.map(s=>s.rows[i][key]);return [Math.min(...a),Math.max(...a)];});}
function soften(id,selector){const root=el(id);root.querySelectorAll(selector).forEach(node=>{node.style.filter='blur(2.2px)';});}
// Split each annual cohort at the year boundary without double-counting its yield.
function calendarSegments(rows){
    const segments=[];
    rows.forEach((r,i)=>{
        if(r.start===null)return;
        const last=r.start+r.duration-1;
        const wrapped=last>DATA.yearWeeks;
        segments.push({row:r,color:DATA.colors[i],start:r.start,
            length:Math.min(r.duration,DATA.yearWeeks-r.start+1),
            context:wrapped?"Planted this year; harvested next year":"Planting and harvest in this year"});
        if(wrapped)segments.push({row:r,color:DATA.colors[i],start:1,
            length:last-DATA.yearWeeks,
            context:"Carry-in from the previous year's identical planting"});
    });
    return segments;
}
function calendarTrace(rows,ghost,opacity){
    const parts=calendarSegments(rows);
    return {type:'bar',orientation:'h',x:parts.map(p=>p.length),
        base:parts.map(p=>p.start-.5),y:parts.map(p=>p.row.crop),
        width:ghost?.64:.56,marker:{color:parts.map(p=>p.color)},
        opacity:ghost?opacity:1,showlegend:false,
        customdata:parts.map(p=>[p.row.start,p.row.end,p.row.bed,p.row.pounds,p.row.revenue,
            p.context,p.row.endUnwrapped]),
        hoverinfo:ghost?'skip':undefined,
        hovertemplate:ghost?undefined:'<b>%{y}</b><br>%{customdata[5]}<br>Annual planting: week %{customdata[0]}<br>Harvest: week %{customdata[1]}<br>Unwrapped harvest week: %{customdata[6]}<br>Bed-feet: %{customdata[2]:,.1f}<br>Annual production: %{customdata[3]:,.1f} lb<br>Annual revenue: $%{customdata[4]:,.2f}<extra></extra>'};
}
function drawCalendar(s,w){
    const traces=[];const n=w.valid.length;
    for(const neighbor of w.valid){
        if(neighbor.percent===s.percent)continue;
        traces.push(calendarTrace(neighbor.rows,true,Math.min(.13,1.0/Math.max(n,1))));
    }
    const ghostCount=traces.length;
    traces.push(calendarTrace(s.rows,false,1));
const shades=['#edf3f9','#f0f6e9','#fff8e8','#faf0e8','#edf3f9'];const shapes=DATA.seasons.map((season,i)=>({type:'rect',xref:'x',yref:'paper',x0:season.start,x1:season.end,y0:0,y1:1,fillcolor:shades[i],line:{width:0},layer:'below'}));
for(const season of DATA.seasons.slice(1))shapes.push({type:'line',xref:'x',yref:'paper',x0:season.start,x1:season.start,y0:0,y1:1,line:{color:'#9cae98',width:1},layer:'above'});
const annotations=DATA.seasons.map(season=>({x:(season.start+season.end)/2,y:1.045,xref:'x',yref:'paper',text:season.name,showarrow:false,font:{size:13,color:'#38553c'},yanchor:'bottom'}));
return Plotly.react('calendar',traces,layout({height:850,barmode:'overlay',xaxis:{title:{text:'Week of model year',standoff:15},range:[.5,52.5],tick0:1,dtick:2,fixedrange:false,gridcolor:'#e2e9df',zeroline:false},yaxis:axisY(),shapes,annotations}),config).then(()=>{el('calendar').querySelectorAll('.barlayer .trace').forEach((node,i)=>{if(i<ghostCount)node.style.filter='blur(2px)';});});}
function drawImpact(s,w,key){const bands=extrema(w.valid,key),xs=DATA.crops.map((_,i)=>i),max=DATA.scales[key];const values=s.rows.map(r=>r[key]);const traces=[{type:'bar',x:xs,y:bands.map(b=>b[1]-b[0]),base:bands.map(b=>b[0]),width:.84,marker:{color:DATA.colors},opacity:.3,customdata:bands,hovertemplate:'%{customdata[0]:,.2f}–%{customdata[1]:,.2f}<extra>Demand-window range</extra>'},{type:'bar',x:xs,y:values,width:.48,marker:{color:DATA.colors},customdata:DATA.crops,hovertemplate:'<b>%{customdata}</b><br>%{y:,.2f} '+metrics[key][1]+'<extra>Selected demand</extra>'},{type:'scatter',mode:'markers',x:xs.flatMap(x=>[x,x]),y:bands.flatMap(b=>b),marker:{symbol:'line-ew',size:10,color:'#455a47',line:{width:1.2,color:'#455a47'}},hoverinfo:'skip'}];
return Plotly.react('impact',traces,layout({height:650,barmode:'overlay',margin:{l:90,r:35,t:25,b:195},xaxis:{tickmode:'array',tickvals:xs,ticktext:DATA.crops,tickangle:-55,range:[-.7,25.7],automargin:true},yaxis:{title:{text:metrics[key][1]},range:[0,max],gridcolor:'#e2e9df',zeroline:false}}),config).then(()=>soften('impact','.barlayer .trace:first-child'));}
function drawCapacity(s,w){const xs=Array.from({length:52},(_,i)=>i+1),lo=xs.map((_,i)=>Math.min(...w.valid.map(v=>v.weekly[i]))),hi=xs.map((_,i)=>Math.max(...w.valid.map(v=>v.weekly[i])));const traces=[{type:'scatter',x:xs,y:lo,mode:'lines',line:{width:0},hoverinfo:'skip'},{type:'scatter',x:xs,y:hi,mode:'lines',line:{width:0},fill:'tonexty',fillcolor:'rgba(95,142,75,.22)',hoverinfo:'skip'},{type:'scatter',x:xs,y:s.weekly,mode:'lines',line:{color:'#47733c',width:2.5},customdata:s.weekly.map(v=>DATA.capacity-v),hovertemplate:'Week %{x}<br>Occupied: %{y:,.1f} bed-feet<br>Unused: %{customdata:,.1f} bed-feet<extra></extra>'},{type:'scatter',x:[1,52],y:[DATA.capacity,DATA.capacity],mode:'lines',line:{color:'#aa6348',dash:'dash',width:2},hovertemplate:'Capacity: %{y:,.0f} bed-feet<extra></extra>'}];return Plotly.react('capacity',traces,layout({height:350,margin:{l:90,r:35,t:35,b:65},xaxis:{title:{text:'Week of model year'},range:[1,52],tick0:1,dtick:2},yaxis:{title:{text:'Occupied bed-feet'},range:[0,DATA.capacity*1.12],gridcolor:'#e2e9df'},annotations:[{x:52,y:DATA.capacity,xref:'x',yref:'y',text:'Farm capacity',showarrow:false,xanchor:'right',yshift:14,font:{color:'#935037'}}]}),config).then(()=>soften('capacity','.scatterlayer .trace:nth-child(2) .js-fill'));}
function drawTrend(s,w,key){const vals=DATA.scenarios.map(v=>v.ok?v.total[key]:null);const traces=[{type:'scatter',x:DATA.scenarios.map(v=>v.percent),y:vals,mode:'lines',connectgaps:false,line:{color:'#47733c',width:2.5},hovertemplate:'%{x}% demand<br>%{y:,.2f} '+metrics[key][1]+'<extra></extra>'},{type:'scatter',x:[s.percent],y:[s.total[key]],mode:'markers',marker:{size:12,color:'#213e2e'},hovertemplate:'Selected: %{x}%<br>%{y:,.2f}<extra></extra>'}];return Plotly.react('trend',traces,layout({height:350,margin:{l:100,r:35,t:25,b:65},xaxis:{title:{text:'Required historical distribution (%)'},range:[0,DATA.maxDemand],dtick:10},yaxis:{title:{text:metrics[key][0]+' ('+metrics[key][1]+')'},rangemode:'tozero',automargin:true},shapes:[{type:'rect',xref:'x',yref:'paper',x0:w.low,x1:w.high,y0:0,y1:1,fillcolor:'rgba(95,142,75,.12)',line:{width:0},layer:'below'}]}),config);}
function table(s){const columns=[['Crop','crop'],['Plant week','start'],['Harvest week (1–52)','end'],['Harvest year offset','harvestCycle'],['Bed-feet','bed'],['Required lb','demand'],['Produced lb','pounds'],['Surplus lb','surplus'],['Nutrition','nutrition'],['Water gal','water'],['Revenue $','revenue'],['Labor h','labor'],['Water efficiency input','efficiency']];el('table').innerHTML='<table><thead><tr>'+columns.map(c=>'<th>'+c[0]+'</th>').join('')+'</tr></thead><tbody>'+s.rows.map(r=>'<tr>'+columns.map((c,i)=>'<td>'+(i?format(r[c[1]],2):escapeHtml(r.crop))+'</td>').join('')+'</tr>').join('')+'</tbody></table>';}
let busy=false,pending=false;
async function render(){if(busy){pending=true;return;}busy=true;try{const objective=el('objective').value;DATA={...PACKAGE,objective,scenarios:PACKAGE.runs[objective]};el('priority').textContent='Optimization priority: '+(objective==='nutrition'?'Nutrition':'Poundage')+' · 1% demand steps · sensitivity grouped into 5-point intervals';const p=+el('demand').value,s=DATA.scenarios.find(v=>v.percent===p),w=rangeWindow(p),key=el('metric').value;el('demandValue').textContent=p+'%';el('window').textContent='Sensitivity: '+w.low+'–'+w.high+'%';el('demand').setAttribute('aria-valuetext',p+' percent demand');if(!s.ok){el('status').textContent='No optimal baseline at '+p+'% demand for '+objective+' (solver status '+s.status+'). Charts are hidden; move the slider to another scenario.';for(const id of ['calendar','impact','capacity','trend','cards','table'])el(id).style.display='none';return;}for(const id of ['calendar','impact','capacity','trend','cards','table'])el(id).style.display='';
el('status').textContent='Each crop must meet '+p+'% of its historical distribution. Sensitivity window: '+w.low+'–'+w.high+'% ('+w.valid.length+' of '+w.all.length+' scenarios optimal).';
el('solverNote').textContent='Selected solve: optimal within solver tolerance; achieved MIP gap '+format(s.gap*100,5)+'%. Main solver target gap '+DATA.solverGap*100+'%.';
el('metricSubtitle').textContent=(objective==='nutrition'?'Nutrition':'Poundage')+' priority · '+metrics[key][0]+' · '+metrics[key][1]+' · consistent vertical scale across demand selections.';el('impactCaption').textContent='Crop names label each bar. Solid bars show '+p+'% demand; the soft bands span the minimum and maximum '+metrics[key][1]+' across all solved demands from '+w.low+'% to '+w.high+'%. This is an input-sensitivity range, not statistical uncertainty.';
el('cards').innerHTML=['pounds','nutrition','water','revenue','labor'].map(k=>{const a=w.valid.map(v=>v.total[k]);return '<div class="card"><div>'+metrics[k][0]+'</div><div class="value">'+(k==='revenue'?'$':'')+format(s.total[k],0)+'</div><div>'+metrics[k][1]+'</div><div class="range">Demand-window range<br>'+format(Math.min(...a),0)+' – '+format(Math.max(...a),0)+'</div></div>';}).join('');table(s);await Promise.all([drawCalendar(s,w),drawImpact(s,w,key),drawCapacity(s,w),drawTrend(s,w,key)]);
}catch(error){el('status').textContent='Chart error: '+error.message;el('status').classList.add('error');console.error(error);}finally{busy=false;if(pending){pending=false;render();}}}
for(const [key,value]of Object.entries(metrics))el('metric').add(new Option(value[0],key));el('metric').value='bed';el('objective').value=DATA.objective;el('demand').max=DATA.maxDemand;el('maximum').textContent=DATA.maxDemand+'%';el('seasonDefinition').textContent=DATA.year+' meteorological seasons: winter Dec–Feb, spring Mar–May, summer Jun–Aug, fall Sep–Nov. Week 1 starts '+DATA.weekOneStart+'; 52 seven-day periods cover 364 days (not ISO weeks).';el('demand').oninput=render;el('metric').onchange=render;el('objective').onchange=render;el('print').onclick=()=>window.print();
render().then(()=>{el('trend').on('plotly_click',event=>{const p=Math.round(event.points[0].x);if(p>=0&&p<=DATA.maxDemand){el('demand').value=p;render();}});});
</script></body></html>
"""

output_folder = Path(__file__).resolve().parent if '__file__' in globals() else Path.cwd()
filename = output_folder / 'lafarm_interactive_critical_demand.html'
html = HTML_TEMPLATE.replace('__PAYLOAD__', json.dumps(payload, allow_nan=False).replace('<', '\\u003c'))
html = html.replace('__PLOTLY_LIBRARY__', get_plotlyjs())
filename.write_text(html, encoding='utf-8')
print(f'\nInteractive dashboard saved to: {filename}', flush=True)
if OPEN_BROWSER:
    webbrowser.open(filename.as_uri())
