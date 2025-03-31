```python
python run_advanced_colight_dsi.py

* 'is_test': True,  # control is train or test

*"inference_epoch": 50, # training epoch of diffusion

*'is_inference': True, # is inference with diffusion

*"NOISE_SCALE": 4,  #  noise scale

*"NOISE_TYPE": 3, # noise type guassion  0， uniform 1，qmin 2，action_diff 3

*"NOISE_LEVEL": 0, # 0 is data noise，1 is data missing

Other baseline experiments in  other_experiments.pdf

Detechnical theory of improve diffusion to denoise and demask in detail_tech.pdf
