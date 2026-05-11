/*:
 * @target MZ
 * @plugindesc [核心前置] 地图即时战斗基础接口（玩家攻击入口 / 普攻技能调用 / 冷却控制）
 * @author CFpanda
 *
 *
 * @param AttackKey
 * @text 攻击键
 * @type number
 * @min 0
 * @default 65
 * @desc 玩家普通攻击的按键。默认 65 = A 键,可修改。
 *
 * @help
 * ============================================================================
 * ■ 按键参数
 * ============================================================================
 * 默认普通攻击键：
 *  65 = 键盘 A 键
 * 若想换其他按键，可以修改参数
 *
 * 常见键值参考：
 *   66  = B
 *   67  = C
 *   68  = D
 *   69  = E
 *   70  = F
 *   81  = Q
 *   82  = R
 *   83  = S
 *   87  = W
 * ============================================================================
 * ■ 插件简介
 * ============================================================================
 * CF_Attack 是地图即时战斗系统中的“攻击入口前置插件”。
 *
 * 本插件主要负责：
 *   1. 监听玩家攻击按键
 *   2. 确定玩家当前普通攻击所使用的技能
 *   3. 调用统一地图技能释放接口
 *   4. 控制基础攻击冷却
 *
 * 它的定位是：
 *   ● 给玩家提供一个“在地图上发动攻击”的统一入口
 *   ● 为后续敌人系统、伤害系统、动画系统提供基础支撑
 *
 * ============================================================================
 * ■ 当前实现的功能
 * ============================================================================
 * 1. 玩家按键攻击
 *    - 默认将键盘 A 键映射为攻击键
 *    - 玩家在地图中按下 A 键时，会尝试执行一次普通攻击
 *
 * 2. 普攻技能自动读取
 *    - 自动读取“队伍队长当前武器”的备注
 *    - 通过武器备注决定普通攻击所对应的技能 ID
 *
 * 3. 地图技能统一调用
 *    - 攻击不会直接写死伤害逻辑
 *    - 会优先调用：
 *        $gamePlayer.useMapSkill(skillId, -1)
 *    - 也就是说，本插件本质上是“攻击输入层 + 技能调用层”
 *
 * 4. 攻击冷却
 *    - 玩家每次普通攻击后会进入短暂冷却
 *
 * 5. 技能范围读取
 *    - 支持从技能备注中读取攻击距离
 *    - 可供其他插件或扩展逻辑使用
 *
 * 6. 前方敌人直线搜索接口
 *    - 提供“按玩家朝向，在前方若干格内寻找敌人事件”的接口
 *    - 供后续战斗插件扩展命中、近战判定、碰撞逻辑使用
 *
 * 7. 基础伤害计算接口
 *    - 提供一个基础的技能伤害计算函数
 *    - 包含命中、闪避、暴击、伤害公式计算
 *    - 主要用于后续插件调用或二次开发
 *
 * ============================================================================
 * ■ 本插件负责什么
 * ============================================================================
 * 本插件负责：
 *   ● 攻击按键输入
 *   ● 普攻技能识别
 *   ● 技能释放入口调用
 *   ● 攻击冷却控制
 *   ● 基础接口提供
 *
 * ============================================================================
 * ■ 本插件不负责什么
 * ============================================================================
 * 本插件不直接负责：
 *   ● 敌人 AI
 *   ● 敌人自动索敌
 *   ● 受击硬直
 *   ● 飘字显示
 *   ● 动画表现
 *   ● 完整技能执行流程
 *   ● 持续伤害 / 击退 / 状态异常等高级战斗效果
 *
 * 上述内容应由后续战斗插件继续实现。
 *
 * ============================================================================
 * ■ 武器备注
 * ============================================================================
 * 在武器备注中写入：
 *
 *   <normalAttackSkill:1>
 *
 * 含义：
 *   当前装备该武器时，玩家普通攻击将使用 1 号技能。
 *
 * 说明：
 *   ● 本插件只读取“队长当前主武器”的备注
 *   ● 如果角色没有装备武器，则默认普通攻击技能为 1
 *   ● 如果装备了武器，但武器没有写此备注，则不执行普通攻击
 *
 * ============================================================================
 * ■ 技能备注
 * ============================================================================
 * 本插件当前支持以下技能备注：
 *
 * 1. 攻击距离
 *    <range:x>
 *
 *    例：
 *    <range:3>
 *
 *    含义：
 *    该技能的判定距离为前方 3 格。
 *
 *    说明：
 *    若技能未填写该备注，则默认距离为 1 格。
 *
 * ----------------------------------------------------------------------------
 * 2. 冷却备注（预留/扩展说明）
 *    <cooldown:x>
 *
 *    例：
 *    <cooldown:30>
 *
 *    说明：
 *    当前版本说明中保留该写法，适合作为后续扩展使用。
 *    但就这份代码本身而言，实际攻击冷却主要由玩家的
 *    _attackCooldown / _attackCooldownMax 控制，
 *    并没有完整从技能备注中自动套用 cooldown 数值。
 *
 * ============================================================================
 * ■ 攻击判定说明
 * ============================================================================
 * 当前插件提供的前方敌人搜索逻辑为：
 *   ● 按玩家当前朝向
 *   ● 从第 1 格开始向前搜索
 *   ● 最多搜索到技能 range 指定的距离
 *   ● 找到第一个满足条件的敌人事件后返回
 *
 * 敌人事件判定条件：
 *   ● 事件对象存在
 *   ● 具有 _enemyEntity
 *   ● _hp > 0
 *
 * 这部分逻辑主要是给后续地图战斗插件使用，
 * 例如近战命中、刺击、前方范围攻击等。
 *
 * ============================================================================
 * ■ 攻击触发条件
 * ============================================================================
 * 玩家满足以下条件时，才允许通过按键攻击：
 *
 *   ● 当前没有消息窗口占用
 *   ● 当前没有事件正在执行
 *   ● 当前不在攻击冷却中
 *
 * 若任一条件不满足，则按键攻击不会触发。
 *
 * ============================================================================
 * ■ 以下是核心战斗系统插件 (在此做个介绍)
 * ============================================================================
 *
 *   推荐插件顺序：
 *   1. 敌人实体插件            CF_Enemy.js
 *   2. 本插件                  CF_Attack.js
 *   3. 攻击体伤害动画处理插件  CF_AttackObject.js
 *   4. 玩家武器动作图处理插件  CF_PlayerActionMotion.js
 *   5. 技能释放与武器联动插件  CF_PlayerSkillCast.js
 *   6. 按键技能栏UI窗口插件    CF_BattleSkill_UI.js
 *   7. 敌人视野仇恨和攻击设置  CF_Eventsight.js
 *   8. 敌人掉落处理插件        CF_EnemyLoot.js
 *
 *      可选辅助插件:   地图UI显示玩家HP/MP插件     CF_HP.js 
 *   (非绑定在战斗系统内的)弓的羽箭依赖扩展插件     CF_BowAmmoSystem.js
 *      强大完美的换装系统+武器与战斗图联动插件     CF_Equipment.js
 *      调控玩家与事件图像X,Y位置的辅助型插件       CF_SetPosition
 *      物品获得信息提示(图标+名称)辅助型插件       CF_LootlogScroll
 *      按键物品快捷栏   辅助型插件                 CF_MapToolBar
 *      连击数显示 增加打击感   扩展型插件          CF_BattleComboSystem
 *      还有许多技能与辅助型扩展插件..............
 * ============================================================================
 * ■ 版权说明
 * ============================================================================
 * 本插件仅限：
 *   ● 学习用途
 *   ● 非商业使用
 *
 *   ● 商业使用需获得作者授权。
 * ============================================================================
 */

(() => {

    const pluginName = "CF_Attack";
    const params = PluginManager.parameters(pluginName);
    const attackKey = Number(params["AttackKey"] || 65);

    //==================================================
    // 0. 按键
    //==================================================
    Input.keyMapper[attackKey] = "attack";

    //==================================================
    // 1. 初始化
    //==================================================
    const _Game_Player_initialize = Game_Player.prototype.initialize;
    Game_Player.prototype.initialize = function() {
        _Game_Player_initialize.call(this);
        this._attackCooldown = 0;
        this._attackCooldownMax = 30;
    };

    //==================================================
    // 2. update
    //==================================================
    const _Game_Player_update = Game_Player.prototype.update;
    Game_Player.prototype.update = function(sceneActive) {
        _Game_Player_update.call(this, sceneActive);
        this.updateAttackByInput();
        this.updateAttackCooldown();
    };

    Game_Player.prototype.updateAttackCooldown = function() {
        if (this._attackCooldown > 0) {
            this._attackCooldown--;
        }
    };

    Game_Player.prototype.canAttackByInput = function() {
        if ($gameMessage.isBusy()) return false;
        if ($gameMap.isEventRunning()) return false;
        if (this._attackCooldown > 0) return false;
        return true;
    };

    Game_Player.prototype.updateAttackByInput = function() {
        if (!this.canAttackByInput()) return;

        if (Input.isTriggered("attack")) {
            this.attackFrontEnemy();
        }
    };

    //==================================================
    // 3. 普通攻击技能
    //==================================================
Game_Player.prototype.normalAttackSkill = function() {
    const skillId = this.normalAttackSkillId();
    return skillId > 0 ? $dataSkills[skillId] : null;
};

    Game_Player.prototype.attackCooldownLeft = function() {
        return this._attackCooldown || 0;
    };

    //==================================================
    // 4. 前方坐标
    //==================================================
    Game_Player.prototype.skillRange = function(skill) {
        if (!skill) return 1;
        const match = skill.note.match(/<range:(\d+)>/i);
        return match ? Number(match[1]) : 1;
    };

    Game_Player.prototype.enemyEventInLine = function(range) {
        for (let i = 1; i <= range; i++) {
            let x = this.x;
            let y = this.y;

            if (this.direction() === 2) y += i;
            else if (this.direction() === 4) x -= i;
            else if (this.direction() === 6) x += i;
            else if (this.direction() === 8) y -= i;

            const events = $gameMap.eventsXy(x, y);
            for (const ev of events) {
                if (ev && ev._enemyEntity && ev._hp > 0) {
                    return ev;
                }
            }
        }
        return null;
    };

    //==================================================
    // 5. 伤害计算（原样保留）
    //==================================================
    Game_Player.prototype.calcSkillDamageToEnemy = function(skill, enemyEvent) {
        const actor = $gameParty.leader();
        if (!actor || !enemyEvent || !skill) return null;

        const enemy = enemyEvent._enemyEntity;

        const hit = (skill.successRate / 100) * (actor.hit || 0.95);
        const eva = enemy.eva || 0.05;

        if (Math.random() > Math.max(0.05, hit - eva)) {
            return { missed: true, damage: 0, critical: false };
        }

        let damage = 0;
        let critical = false;

        if (skill.damage && skill.damage.type > 0) {
            const a = actor;
            const b = enemy;
            const v = $gameVariables._data;

            try {
                damage = Math.floor(eval(skill.damage.formula));
            } catch (e) {
                damage = 0;
            }

            if (skill.damage.type === 1) {
                critical = Math.random() < (actor.cri || 0);
                if (critical) damage *= 1.5;
            }

            damage = Math.max(0, damage);
        }

        return { missed: false, damage, critical };
    };

    //==================================================
    // 6. 攻击
    //==================================================
Game_Player.prototype.normalAttackSkillId = function() {
    const actor = $gameParty.leader();
    if (!actor) return 0;

    const weapon = actor.weapons()[0];
    // 没武器时，默认普通攻击技能为 1
    if (!weapon) return 1;
    // 有武器但没备注时，不释放普通攻击
    if (!weapon.note) return 0;

    const match = weapon.note.match(/<normalAttackSkill:(\d+)>/i);
    return match ? Number(match[1]) : 0;
};

Game_Player.prototype.attackFrontEnemy = function() {
    const skillId = this.normalAttackSkillId();
    if (skillId <= 0) return false;

    // 直接复用 QWER 的统一技能释放逻辑
    if (this.useMapSkill) {
        return this.useMapSkill(skillId, -1);
    }

    return false;
};

    //==================================================
    // 7. MZ 插件指令
    //==================================================
    PluginManager.registerCommand("CF_Attack", "PlayerAttack", () => {
        $gamePlayer.attackFrontEnemy();
    });

})();
