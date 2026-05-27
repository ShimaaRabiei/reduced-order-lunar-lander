# Reduced-Order LunarLander

This folder contains a reduced-order version of the Gymnasium Box2D LunarLander task. The model is useful for experiments where the policy commands an attitude reference instead of directly commanding the lateral booster.

The original continuous LunarLander action is `[main engine, lateral booster]`. In this version, the action is

```text
[main engine, theta_star]
```

where `theta_star` is a commanded lander attitude. The rotational dynamics are abstracted away during training.

## Model

At each step, the reduced model sets

```text
theta = theta_star
angular_velocity = 0
```

The commanded attitude changes the direction of the main-thrust impulse. The side-thruster block is not used in the reduced model, so there is no lateral-thruster force, torque, or fuel penalty during reduced-order training.

## Observation

The default observation is

```text
[x, y, vx, vy, theta_star_previous, left_leg_contact, right_leg_contact]
```

The previous attitude command is included because the training objective can penalize changes in the attitude reference.

## Action

The action space is

```text
Box(-1, 1, shape=(2,), dtype=float32)
```

The first action uses the same main-engine convention as continuous LunarLander: negative values turn the main engine off, and nonnegative values scale the engine between 50% and 100% power.

The second action is mapped to the commanded attitude as

```text
theta_star = theta_limit * action[1]
```

The default value is `theta_limit = 35 degrees`.

## Reward

The task reward follows the LunarLander shaping structure: position, velocity, attitude, leg contacts, main-engine fuel, and terminal landing/crash reward. The side-thruster fuel term is removed because the side thruster is not part of the reduced-order action.

The training reward is

```text
training_reward = task_reward - lambda * abs(theta_star_t - theta_star_{t-1})
```

Changing `lambda` controls how strongly the policy is encouraged to produce smoother attitude-reference sequences.

## Reset and evaluation

Training uses the original LunarLander reset mechanism.

For final evaluation, the script generates a fixed set of stock reset seeds and reuses them across runs. This gives a fair comparison across different values of `lambda`.

The default final evaluation uses 333 fixed reset seeds.

## Termination

The reduced model keeps the stock LunarLander termination logic:

```text
body contact with terrain -> crash
absolute x position outside the viewport -> failure
Box2D body is not awake -> successful landing
```

The success condition is still based on `not lander.awake`. It is a Box2D sleep state, not a rule imposed from the throttle or from `theta_star`.

## Quick test

```powershell
python train_reduced_order_lunar_lander.py --mode train --variation_lambda 0.0 --total_steps 8192 --num_envs 2 --steps_per_rollout 1024 --eval_every_rollouts 1 --eval_episodes 5 --final_eval_episodes 10 --device cpu
```

## Training

Without attitude-variation penalty:

```powershell
python train_reduced_order_lunar_lander.py --mode train --variation_lambda 0.0 --total_steps 500000 --num_envs 8 --steps_per_rollout 2048 --final_eval_episodes 333
```

With attitude-variation penalty:

```powershell
python train_reduced_order_lunar_lander.py --mode train --variation_lambda 0.25 --total_steps 500000 --num_envs 8 --steps_per_rollout 2048 --final_eval_episodes 333
```

A simple sweep is

```text
lambda = 0.0, 0.1, 0.25, 0.5, 1.0
```

## Outputs

Each run saves the best checkpoint, the last checkpoint, training history, fixed evaluation seeds, final evaluation summaries, and final trajectories.

## Credit

This example is based on the Gymnasium Box2D LunarLander implementation.
