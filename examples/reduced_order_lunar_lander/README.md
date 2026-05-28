# Reduced-Order LunarLander

This example defines a reduced-order version of the Gymnasium Box2D LunarLander task. The model keeps the translational motion, terrain, leg contacts, main-engine impulse, and official LunarLander terminal logic, but abstracts away the rotational dynamics.

The policy outputs a main-engine command and a commanded attitude reference. The commanded attitude is imposed directly in the reduced model, so the agent does not learn the rotational dynamics during training.

## Reduced model

The reduced action is

```text
a_R = [main_cmd, theta_star_norm]
```

where `main_cmd` follows the continuous LunarLander main-engine convention and `theta_star_norm` is mapped to

```text
theta_star = theta_limit_rad * theta_star_norm
```

The baseline setup used here sets `theta_limit_deg = 20`.

The reduced observation is

```text
[x, y, vx, vy, theta_star_previous, left_contact, right_contact]
```

The previous attitude reference, `theta_star_previous`, is included because the training objective can include a reference-variation cost.

The reduced model imposes

```text
theta = theta_star
angular_velocity = 0
```

at each step. The side thruster is not part of the reduced action and is not fired in the reduced model. Translation is affected by the commanded attitude through the direction of the main-thrust impulse.

## Reward and training objective

The task reward follows the official LunarLander shaping terms for distance to the pad, velocity, attitude, leg contacts, main-engine fuel cost, and terminal success/crash reward. In the reduced model, the attitude term is evaluated using the commanded/imposed attitude `theta_star`, since the rotational dynamics are abstracted away. Since the side thruster is not part of the reduced action, no side-thruster fuel cost is used in the reduced model.

The training reward is

```text
train_reward = task_reward - variation_lambda * |theta_star_t - theta_star_{t-1}| - step_penalty
```

The step penalty is used to discourage indefinite hovering. It does not replace the task reward and it does not change the official success condition.

## Training and evaluation

Training uses the official LunarLander reset mechanism. Evaluation uses fixed reset seeds so different training runs can be compared on the same initial conditions.

The `variation_lambda = 0` baseline was trained in two stages: a cold PPO run followed by a warm fine-tuning run with a smaller learning rate and smaller exploration noise.

The reported policy is the final model from training, not a best checkpoint selected during training.

## Lambda-zero baseline result

```text
variation_lambda: 0.0
theta_limit_deg: 20
step_penalty: 0.02
evaluation episodes: 333

success_rate: 0.5225
crash_rate: 0.1021
out_of_bounds_rate: 0.0060
mean_task_return: 139.6360
mean_train_return: 125.5874
mean_variation: 2.9319
mean_length: 702.4294
mean_final_speed: 0.0131
mean_final_both_legs_contact: 0.7447
mean_final_lander_awake: 0.4775
landing_candidate_rate: 0.4474
```

## Landing videos

Five successful landing videos from the `variation_lambda = 0` warm baseline are saved in:

```text
media/videos/lam0_theta20_step002_warm/
```

The videos were generated from fixed evaluation reset seeds using the saved warm baseline model.

## Repository files

```text
train_reduced_order_lunar_lander.py
record_reduced_order_lunar_lander_videos.py
commands/lam0_training_commands.md
models/lam0_theta20_step002_cold/final_model.pt
models/lam0_theta20_step002_cold/config.json
models/lam0_theta20_step002_cold/training_history.csv
models/lam0_theta20_step002_warm/final_model.pt
models/lam0_theta20_step002_warm/config.json
models/lam0_theta20_step002_warm/training_history.csv
results/lam0_theta20_step002_warm/eval_333_summary.json
media/videos/lam0_theta20_step002_warm/
```
