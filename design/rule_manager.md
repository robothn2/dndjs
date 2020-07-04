# DND rules
《龙与地下城》的规则相当复杂，核心是攻击方和防御方的数值对抗，但数值的计算比起普通游戏就要麻烦多了。下面使用近战攻击命中规则来说明一下计算过程(感谢 fandom wiki)：
- 攻击方 AB(Attack Bonus) [计算规则](https://nwn.fandom.com/wiki/Attack_bonus): 各部分之间为累加
  - d20骰子  一个介于 1 - 20 之间的随机数值
  - 基本攻击加值(Base Attack Bonus)
  - 力量调整值(Strength Modifier)
  - 各种专长(Feat)加值
  - 武器强化值(Enhancement)
  - 效果加值(Effect)，一般由法术导致
  - 对方状态  跌倒、骑乘等状态
  - 其它
  - 攻击类型  近战(Melee), 远程(Ranged), 接触(Touched)
- 防守方 AC(Armor Class) [计算规则](https://nwn.fandom.com/wiki/Armor_class): 各部分之间为累加
  - 初始防御 10
  - 天生防御等级(Natural Bonus)，只取最大值
  - 护甲加值(Armor Bonus) 由装备栏的护甲槽内的护甲、法术提供，只取最大值
  - 盾牌加值(Shield Bonus) 一般由装备栏的副手槽内的盾牌、法术提供，只取最大值
  - 偏斜加值(Deflection Bonus) 由各种装备物品和法术提供，只取最大值
  - 闪躲加值(Dodge Bonus) 由各种装备物品和法术提供，可叠加，但最大 20
  - 其它加值(Other) 由能力、法术、特长等提供，可叠加
- 对抗规则
  - 当攻击方 d20 投出 1 时，直接判定攻击失手(Miss)
  - 当攻击方 d20 投出 20 时，直接判定攻击命中
  - 当攻击方 d20 投出其它值时，若 DC >= AC，判定成功命中，否则为失手
  
# 如何计算上述规则
- 首先想到的方案是：分别使用函数实现各种中间值计算，然后累加
  - 当你发现很多中间值可以缓存来加速计算时，你就需要增加缓存失效时的标志和相应处理，管理这些标志和依赖关系十分麻烦
  - 当你需要改用不同规则时，上面这些关系都需要重新实现
- 基于上述需求，抽象一套支持缓存、递归、lambda算子的规则管理器
  - Source 将 feat，race，class，buff 等分解为多个 Source
  - Calculator 实现计算规则，包含多个 Source 和多个上游 Calculator。计算规则需要提供两个方法：MethodSource和MethodMerge
    - MethodSource 用于处理每个 Source/上游Calculator 的返回值，默认为原值返回
    - MethodMerge 用于合并两个 Source/上游Calculator 的 MethodSource 返回值，默认为累加
    - Calculator 支持缓存上次返回的值
  - SourceManager 管理的 Calculator 树，随每种 DND 规则而拥有不同结构定义

