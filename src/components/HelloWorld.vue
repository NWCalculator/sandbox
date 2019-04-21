<template>
  <div class="calculator">
    <div class="column">
      <div class="field">
        <label class="label" for>Level</label>
        <input type="number" v-model.number="level" min="1" max="60">
        <button @click="level = 60">MAX</button>
      </div>
      <div class="field">
        <label class="label" for>Attributes</label>
        {{totalAttributePoints}}
      </div>
      <div class="field">
        <label class="label" for>Spent Attributes</label>
        {{spentAttributePoints}}
      </div>
      <div class="field">
        <div class="equip-load">
          <div class="equip-total">Equip Load: {{this.equipLoad}} / {{this.equipLoadTotal}}</div>
          <div class="equip-load-bar-empty">
            <div class="equip-fast"></div>
            <div class="equip-normal"></div>
            <div class="equip-slow"></div>
            <div :class="equipLoadBarStyles" :style="{ width: equipBarProgress + '%'}"></div>
          </div>
        </div>
      </div>
      <fieldset class="field">
        <div class="attribute" v-for="(attribute, key) in attributes" :key="key">
          <label for class="label">{{key}}</label>
          <input type="number" min="5" v-model.number="attributes[key]">
        </div>
      </fieldset>
      <div class="field">
        <button class="reset" @click.prevent="reset">RESET</button>
        <button class="button" @click.prevent="increment">Add Weight</button>
        <button class="button" @click.prevent="decrement">Subtract Weight</button>
      </div>
    </div>
    <div class="column">
      <div class="attributes">
        <div class="attribute">
          <span>Health:</span>
          <span>{{health}}</span>
        </div>
        <div class="attribute">
          <span>Stamina:</span>
          <span>{{stamina}}</span>
        </div>
        <div class="attribute">
          <span>Mana:</span>
          <span>{{mana}}</span>
        </div>
        <div class="attribute">
          <span>Equip Load:</span>
          <span>{{equipLoadTotal}}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const calc = function(attribute, preReq, milestone, modifier, multiplier) {
  return attribute <= preReq
    ? 0
    : attribute <= milestone
    ? modifier * (attribute - preReq)
    : modifier * multiplier;
};

const sum = function(base, attribute, ...args) {
  return args.reduce(
    (total, [preReq, milestone, modifier, multiplier]) =>
      total + calc(attribute, preReq, milestone, modifier, multiplier),
    base
  );
};

export default {
  name: "Calculator",
  props: {
    msg: String
  },
  data() {
    return {
      level: 1,
      bonusAttributes: 10,
      equipLoad: 0.0,
      attributes: {
        consitution: 5,
        strength: 5,
        endurance: 5,
        physique: 5,
        strength: 5,
        dexterity: 5,
        intellect: 5,
        focus: 5,
        faith: 5
      }
    };
  },

  computed: {
    equipLoadBarStyles() {
      return [
        {
          "is-fast": this.isFast,
          "is-normal": !this.isFast && this.equipBarProgress > 0,
          "is-heavy": this.isHeavy
        },
        "equip-load-bar"
      ];
    },
    under() {
      return this.level <= 40
        ? Math.ceil(this.level !== 1 ? (this.level - 1) * 2 : 0)
        : 78;
    },
    over() {
      return this.level > 40 ? (this.level - 40) * 1 : 0;
    },
    unspectTradeSkillPoints() {
      return this.level * 1;
    },
    totalTradeSkillPoints() {
      return this.unspentTradeSkillPoints - this.spentTradeSkillPoints;
    },
    spentAttributePoints() {
      const total = Object.values(this.attributes).reduce((total, amount) => {
        return amount + total;
      }, 0);
      // 40 being the base amount of attribute points already assigned.
      return total - Object.keys(this.attributes).length * 5;
    },
    totalAttributePoints() {
      return (
        Math.ceil(this.under + this.over + this.bonusAttributes) -
        this.spentAttributePoints
      );
    },

    fastWeight() {
      return 0.301 * this.equipLoadTotal;
    },

    isFast() {
      return this.equipLoad <= this.fastWeight;
    },

    isHeavy() {
      return this.equipLoad > this.equipLoadTotal;
    },
    // isDisabled() {
    //   return this.totalAttributePoints === 0;
    // },

    equipBarProgress() {
      return (this.equipLoad / this.equipLoadTotal) * 100;
    },

    equipLoadTotal() {
      return sum(
        50,
        this.attributes.physique,
        [5, 15, 1, 10],
        [15, 25, 0.8, 10],
        [25, 35, 0.7, 10],
        [35, 45, 0.6, 10],
        [45, 55, 0.5, 10],
        [55, 65, 0.4, 10],
        [65, 75, 0.3, 10],
        [75, 85, 0.2, 10],
        [85, 108, 0.1, 10]
      );
    },

    health() {
      return sum(
        500,
        this.attributes.consitution,
        // [startingLv, milestoneLv, value, delta]
        [5, 10, 11, 5],
        [10, 15, 10, 5],
        [15, 20, 9, 5],
        [20, 30, 8, 10],
        [30, 40, 7, 10],
        [40, 44, 6, 4],
        [44, 50, 5, 6],
        [50, 64, 4, 14],
        [64, 74, 3, 10],
        [74, 85, 2, 11],
        [85, 108, 1, 23]
      );
    },

    stamina() {
      return sum(
        100,
        this.attributes.endurance,
        [5, 15, 4, 10],
        [15, 20, 3, 5],
        [20, 40, 2, 20],
        [40, 108, 1, 68]
      );
    },

    _faith() {
      return sum(
        0,
        this.attributes.faith,
        [5, 15, 4, 10],
        [15, 20, 3, 5],
        [20, 40, 2, 20],
        [40, 108, 1, 68]
      );
    },

    _intellect() {
      return sum(
        0,
        this.attributes.intellect,
        [5, 15, 4, 10],
        [15, 20, 3, 5],
        [20, 40, 2, 20],
        [40, 108, 1, 68]
      );
    },

    mana() {
      return 100 + (this._faith + this._intellect);
    },

    resFrostbite() {
      return sum(
        0,
        this.attributes.consitution,
        [5, 25, 2, 20],
        [25, 60, 1, 40],
        [60, 108, 0, 48]
      );
    }
  },

  methods: {
    reset() {
      Object.keys(this.attributes).forEach(key => (this.attributes[key] = 5));
    },
    increment() {
      this.equipLoad += 0.5;
    },

    decrement() {
      this.equipLoad -= 0.5;
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.calculator {
  display: flex;
}
.column {
  flex: 1 1 auto;
  padding: 0 1rem;
}
.field,
.attribute {
  margin: 1.5rem 0;
  display: flex;
  align-items: center;
}

.label {
  flex: 1;
}

.attributes {
  margin-top: 1.5rem;
  border: 1px solid #000;
  padding: 1rem;
}

.equip-load {
  display: flex;
  flex-direction: column;
  width: 100%;
}

.equip-total {
  align-self: flex-end;
}

.equip-load-bar-empty {
  position: relative;
  display: flex;
  width: 100%;
  height: 10px;
  background: rgba(55, 55, 55, 0.7);
}

.equip-load-bar {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 1;
  height: 10px;
}

.is-fast {
  background-color: lightgreen;
}

.is-normal {
  background-color: darkgoldenrod;
}

.is-heavy {
  background-color: yellow;
}

.equip-fast {
  flex-basis: 30.1%;
  width: 30.1%;
  border-right: 1px solid #000;
}
.equip-normal {
  flex-basis: 50%;
  width: 50%;
  border-right: 1px solid #000;
}
.equip-heavy {
  flex-basis: 20%;
  width: 20%;
  border-right: 1px solid #000;
}
</style>
