# Raw Tactical Attributes for Match-Level Soccer Style Analysis

## 1. Overview

This submission creates 50 interpretable match-team attributes from SkillCorner Dynamic Events. The output file contains one row per team per match, so the 10-match sample produces 20 rows. The goal is not to predict match results. The goal is to describe how each team attacked, progressed the ball, occupied territory, increased tempo, pressed, used set pieces, and created support around the ball.

All features follow the competition constraints. They are raw aggregated counts, duration sums, or distance sums. The submission does not use percentages, ratios, per-minute values, per-possession values, scaled metrics, normalized values, embeddings, or machine-learned outputs.

## 2. Data Loading and Reproducibility

The notebook reads files dynamically from the repository structure:

`opendata/data/matches/[match_id]/[match_id]_dynamic_events.csv`

The code searches for all files ending in `_dynamic_events.csv`, so match IDs are not hardcoded. For each file, it computes features separately for each `team_id`, combines all match-team rows, sorts by `match_id` and `team_id`, and writes the required file:

`features.csv`

The expected output shape is 20 rows and 52 columns: `match_id`, `team_id`, and 50 feature columns.

## 3. Feature Design Philosophy

The feature set is organized into eight tactical families:

1. Phase duration profile
2. Passes into the attacking third
3. Territorial dominance over time segments
4. Speed of play and tempo proxies
5. Build-up progression profile
6. Pressure and disruption proxies
7. Set-piece vs open-play structural indicators
8. Spatial attack, combination play, and passing sequences before loss

This structure gives a broad description of team behavior without relying on simple box-score statistics. A team can be described as direct, patient, wide, central, transitional, set-piece-heavy, high-tempo, or pressure-oriented based on raw tactical event totals.

## 4. Feature Definitions and Interpretation

### 4.1 Phase Duration Profile

The phase duration features sum the duration of player-possession events inside SkillCorner phase labels such as build-up, create, finish, quick break, transition, direct play, set play, and chaotic play.

Features:

- `phase_build_up_duration_sum`
- `phase_create_duration_sum`
- `phase_finish_duration_sum`
- `phase_quick_break_duration_sum`
- `phase_transition_duration_sum`
- `phase_direct_duration_sum`
- `phase_set_play_duration_sum`
- `phase_chaotic_duration_sum`

These features measure how much raw possession time a team spent in different tactical phases. High build-up duration suggests controlled possession from deeper areas. High quick-break or transition duration suggests fast attacking moments. High set-play duration suggests more attacking activity from dead-ball situations.

### 4.2 Passes Into the Attacking Third

These features count pass-ending player possessions where the target or reception reaches the attacking third.

Features:

- `pass_to_attacking_third_count`
- `forward_pass_to_attacking_third_count`
- `long_pass_to_attacking_third_count`
- `central_pass_to_attacking_third_count`
- `wide_pass_to_attacking_third_count`

These features measure how a team enters advanced territory through passing. Forward and long attacking-third passes describe direct progression. Central attacking-third passes describe penetration through central or half-space lanes. Wide attacking-third passes describe flank progression.

### 4.3 Territorial Dominance Over Time Segments

These features count player-possession actions that end in the attacking third during fixed match time segments.

Features:

- `seg_0_15_att_third_possession_count`
- `seg_15_30_att_third_possession_count`
- `seg_30_45_att_third_possession_count`
- `seg_45_60_att_third_possession_count`
- `seg_60_75_att_third_possession_count`
- `seg_75_120_att_third_possession_count`

These features describe when a team occupied dangerous territory. For example, a high value in the final segment may suggest late pressure, chasing the game, or sustained territorial control near the end.

### 4.4 Speed of Play and Tempo Proxies

The speed-of-play features are raw counts and duration sums that describe quick ball movement and high-tempo possession actions.

Features:

- `one_touch_possession_count`
- `quick_pass_possession_count`
- `carry_possession_count`
- `forward_momentum_possession_count`
- `hsr_possession_count`
- `sprinting_possession_count`
- `high_speed_duration_sum`

These features avoid averages or rates. High one-touch and quick-pass counts suggest fast circulation. High carry count suggests more dribbling or ball-carrying. High high-speed and sprinting possession counts suggest more explosive attacking actions.

### 4.5 Build-up Progression Profile

These features describe whether build-up possession actually advances the ball and whether it connects into more advanced attacking phases.

Features:

- `build_up_forward_possession_count`
- `build_up_forward_distance_sum`
- `build_up_pass_end_count`
- `build_up_to_create_count`
- `build_up_to_finish_count`

High values suggest that the team’s build-up was not only safe circulation, but also progressive. The transition from build-up to create or finish phases indicates that controlled possession moved into attacking structure.

### 4.6 Pressure and Disruption Proxies

These features count defensive engagements and pressure outcomes.

Features:

- `pressing_count`
- `pressure_count`
- `counter_press_count`
- `recovery_press_count`
- `force_backward_count`
- `stop_possession_danger_count`
- `reduce_possession_danger_count`

These features describe out-of-possession behavior. High pressing and counter-pressing counts suggest aggressive defensive pressure. High force-backward, stop-danger, and reduce-danger counts suggest pressure that disrupts the opponent’s possession quality.

### 4.7 Set-Piece vs Open-Play Structural Indicators

These features separate set-piece attacking actions from non-set-play attacking actions.

Features:

- `set_play_possession_count`
- `set_play_shot_count`
- `set_play_goal_count`
- `set_play_penalty_area_end_count`
- `open_play_shot_count`

These features help identify whether a team’s threat came more from set pieces or from open play. For example, high set-play shot count suggests dead-ball attacking threat, while high open-play shot count suggests chance creation through normal possession phases.

### 4.8 Spatial Attack, Combination Play, and Passes Before Loss

These features describe the team’s support structure, use of wide/central space, off-ball movement, and passing sequences before losing possession.

Features:

- `wide_passing_option_count`
- `central_passing_option_count`
- `support_run_count`
- `overlap_underlap_run_count`
- `cross_receiver_run_count`
- `passes_before_loss_total`
- `loss_after_6_plus_pass_count`

Wide passing options and cross-receiver runs describe flank-oriented play. Central passing options describe central and half-space support. Support runs describe short combination structures. Overlap and underlap runs describe coordinated movement to bypass defenders. Passes-before-loss features describe how many pass-ending actions occurred in phases that eventually ended with a ball loss.

## 5. How the Features Are Computed

The notebook filters each match file by `team_id`, then separates rows by `event_type`:

- `player_possession`
- `passing_option`
- `off_ball_run`
- `on_ball_engagement`

Counts are computed by filtering categorical labels or Boolean flags. Duration features are computed by summing the raw `duration` column. Distance features are computed by summing positive forward movement from `x_end - x_start`. Time-segment features use the raw `minute_start` column. The final-third pass features use pass end type plus target or reception location fields.

No modeling step is used. The notebook is deterministic and reproducible.

## 6. Limitations

The dataset contains only 10 matches, so the attributes should be interpreted as match descriptions rather than long-term team identities.

Because the competition requires raw aggregate features only, teams with more total possession time can naturally accumulate more actions, distances, and durations. This is intentional for rule compliance, but it means the features should not be interpreted as efficiency metrics.

The passes-before-loss features use phase-level possession loss indicators. They describe passing inside phases that eventually ended with ball loss, not a normalized possession security score.

The approach depends on the availability and consistency of Dynamic Event labels. If a future file is missing a column, the notebook safely fills the related feature with zero.

## 7. Future Improvements

Future versions could add tracking-based player-region observation counts, such as total player observations in the attacking third, central lane, or penalty area. If normalized features were allowed, future work could also add per-possession or per-minute versions. However, this submission intentionally avoids those values because the rules require raw aggregated attributes only.

Future work could also visualize each tactical family by match, show phase maps, and compare profiles across multiple matches once more data becomes available.