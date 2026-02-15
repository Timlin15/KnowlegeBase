在经历了快一个学期的~~摸鱼~~沉淀后，终于打算开始上手实践了。目前打算是先根据Openvla-oft的训练完的权重在LIBERO benchmark上进行测试，然后调用Openvla-oft中给的lora fine tune命令对openvla进行微调，以及自己手搓一个RL微调的程序，看能否成功。
选择Openvla的原因是[How to Get Started with Embodied AI Research - PKU epic](https://jiangranlv.notion.site/)中提到
> OpenVLA：第一个开源且易于follow的VLA。

而且今年正好有[Fine-Tuning Vision-Language-Action Models: Optimizing Speed and Success | Abstract](https://arxiv.org/abs/2502.19645)论文发表，提供了Optimized Fine-Tuning的一些方案。同时，相比pi0.5，openvla的架构更加简单，更容易上手一些。

所以这次的计划只需要构建rl fine tuning的程序即可，目前~~参考ChatGPT打算根据[drqv2](https://github.com/facebookresearch/drqv2)为框架搭建rl框架~~参考Gemini，因为openvla-oft是一个vla套了一层diffusion，不适合使用为tokenized设计的PPO/GRPO算法，打算使用Filtered BC对diffusion进行微调，冻结vla模型，适配4090的24G显存，或者用grpo/ppo微调vla，8卡A100应该也可以胜任。
