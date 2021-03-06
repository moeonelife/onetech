<template>
  <div class="changeLogCommit">
    <div class="message">
      <div class="messageContent">{{commit.message}}</div>
      <div class="messageDate">{{date}}</div>
      <div v-if="hasChanges" class="collapse" @click="toggleCollapse">
        <span v-if="collapsed">&#x25C0;</span>
        <span v-else>&#x25BC;</span>
      </div>
    </div>

    <div v-if="!collapsed && commit.removedObjectIDs" class="objectsList">
      <h3>移除物体</h3>
      <div class="objects">
        <div v-for="object in removedObjects" class="objectWrapper">
          <div class="object">
            <ObjectImage :object="object" scaleUpTo="80" />
            <div class="name">{{object.name}}</div>
          </div>
        </div>
      </div>
      <div v-if="commit.removedObjectIDs.length > objectLimit" class="showMore">
        <span class="link" @click="showMoreObjects">显示更多...</span>
      </div>
    </div>

    <div v-if="!collapsed && commit.addedObjectIDs" class="objectsList">
      <h3>新增物体</h3>
      <div class="objects">
        <div v-for="object in addedObjects" v-if="object" class="objectWrapper">
          <router-link class="object"
              :to="object.url()"
              :class="{disabled: object.legacy}"
              :event="object.legacy ? '' : 'click'">
            <ObjectImage :object="object" scaleUpTo="80" />
            <div class="name">{{object.name}}</div>
          </router-link>
        </div>
      </div>
      <div v-if="commit.addedObjectIDs.length > objectLimit" class="showMore">
        <span class="link" @click="showMoreObjects">显示更多...</span>
      </div>
    </div>

    <div v-if="!collapsed && commit.removedTransitions">
      <h3>移除配方</h3>
      <div class="transitions">
        <div v-for="(transition, index) in removedTransitions"
            class="transition"
            :key="index">
          <TransitionView :transition="transition" />
        </div>
        <div v-if="commit.removedTransitions.length > 1" class="transitionSpacer"></div>
      </div>
      <div v-if="commit.removedTransitions.length > transitionLimit" class="showMore">
        <span class="link" @click="showMoreTransitions">显示更多...</span>
      </div>
    </div>

    <div v-if="!collapsed && commit.addedTransitions">
      <h3>新增配方</h3>
      <div class="transitions">
        <div v-for="(transition, index) in addedTransitions"
            class="transition"
            :key="index">
          <TransitionView :transition="transition" />
        </div>
        <div v-if="commit.addedTransitions.length > 1" class="transitionSpacer"></div>
      </div>
      <div v-if="commit.addedTransitions.length > transitionLimit" class="showMore">
        <span class="link" @click="showMoreTransitions">显示更多...</span>
      </div>
    </div>

    <div v-if="!collapsed && commit.objectChanges" class="objectChanges">
      <h3>修改物体</h3>
      <div class="objects">
        <div v-for="change in objectChanges"
            class="changedObject"
            :key="change.id">
          <ChangeLogObjectChange :change="change" />
        </div>
      </div>
      <div v-if="commit.objectChanges.length > objectChangeLimit" class="showMore">
        <span class="link" @click="showMoreObjectChanges">显示更多...</span>
      </div>
    </div>
  </div>
</template>

<script>
import GameObject from '../models/GameObject';

import ObjectImage from './ObjectImage';
import TransitionView from './TransitionView';
import ChangeLogObjectChange from './ChangeLogObjectChange';

export default {
  props: [
    'commit',
  ],
  components: {
    ObjectImage,
    TransitionView,
    ChangeLogObjectChange,
  },
  data() {
    return {
      objectLimit: 8,
      transitionLimit: 6,
      objectChangeLimit: 6,
      collapsed: false
    };
  },
  created() {
    if (this.commit.legacyObjects) {
      for (let object of this.commit.legacyObjects)
        GameObject.addLegacyObject(object);
    }
  },
  computed: {
    addedObjects() {
      return this.commit.addedObjectIDs.slice(0, this.objectLimit).map(id => GameObject.find(id));
    },
    removedObjects() {
      return this.commit.removedObjectIDs.slice(0, this.objectLimit).map(id => GameObject.find(id));
    },
    addedTransitions() {
      return this.commit.addedTransitions.slice(0, this.transitionLimit);
    },
    removedTransitions() {
      return this.commit.removedTransitions.slice(0, this.transitionLimit);
    },
    objectChanges() {
      return this.commit.objectChanges.slice(0, this.objectChangeLimit);
    },
    hasChanges() {
      return this.commit.addedObjectIDs
        || this.commit.removedObjectIDs
        || this.commit.addedTransitions
        || this.commit.removedTransitions
        || this.commit.objectChanges;
    },
    date() {
      const date = new Date(this.commit.date);
      const months = [
        "一月", "二月", "三月",
        "四月", "五月", "六月", "七月",
        "八月", "九月", "十月",
        "十一月", "十二月"
      ];
      var month = date.getMonth();
      var day = date.getDate();
      var year = date.getFullYear();
      return `${months[month]} ${day}, ${year}`;
    },
  },
  methods: {
    showMoreObjects() {
      this.objectLimit += 24;
    },
    showMoreTransitions() {
      this.transitionLimit += 24;
    },
    showMoreObjectChanges() {
      this.objectChangeLimit += 24;
    },
    toggleCollapse() {
      this.collapsed = !this.collapsed;
    }
  }
}
</script>

<style scoped>
  .changeLogCommit {
    display: flex;
    flex-direction: column;
    background-color: #222;
    border-radius: 10px;
    margin: 10px 0;
  }

  .changeLogCommit .message {
    display: flex;
    background-color: #333;
    border-radius: 10px;
    margin: 10px;
    padding: 10px 20px;
    align-items: flex-start;
  }

  .changeLogCommit .messageContent {
    flex: 1;
  }

  .changeLogCommit .messageDate {
    padding-left: 20px;
    color: #999;
  }

  .changeLogCommit .collapse {
    color: #ccc;
    padding-left: 10px;
    cursor: pointer;
  }

  .changeLogCommit h3 {
    font-size: 16px;
    color: #ccc;
    margin-top: 5px;
    margin-bottom: 0;
    text-align: center;
  }

  .changeLogCommit .objects {
    display: flex;
    flex-wrap: wrap;
    padding: 0 10px;
    padding-bottom: 10px;
    justify-content: center;
  }

  .changeLogCommit .objectWrapper {
    display: flex;
    align-items: stretch;
    min-width: 150px;
    width: 25%;
  }

  .changeLogCommit .object {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    background-color: #333;
    border-radius: 10px;
    padding: 10px;
    margin: 10px;
    border: 1px solid transparent;
    text-decoration: inherit;
    color: inherit;
  }
  .changeLogCommit a.object:hover {
    border: 1px solid #eee;
    background-color: #222;
  }
  .changeLogCommit a.object.disabled {
    cursor: default;
  }
  .changeLogCommit a.object.disabled:hover {
    border: 1px solid transparent;
    background-color: #333;
  }

  .changeLogCommit .object .name {
    text-align: center;
  }

  .changeLogCommit .object .imgContainer {
    width: 128px;
    height: 128px;
    border: solid 1px transparent;
  }

  .changeLogCommit .changedObject {
    display: flex;
    align-items: stretch;
    min-width: 200px;
    width: 33.3333%;
  }

  .changeLogCommit .transitions {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    padding-bottom: 10px;
  }

  .changeLogCommit .transition {
    width: 380px;
    margin: 0 10px;
  }

  .changeLogCommit .transitionSpacer {
    height: 0;
    width: 380px;
    margin: 0 10px;
  }

  .changeLogCommit .showMore {
    margin-bottom: 10px;
    text-align: center;
  }

  .changeLogCommit .showMore .link {
    text-decoration: none;
    font-style: italic;
    cursor: pointer;
  }

  .changeLogCommit .showMore .link:hover {
    text-decoration: underline;
  }

  @media only screen and (max-width: 768px) {
    .changeLogCommit .messageDate {
      display: none;
    }
    .changeLogCommit .objectWrapper {
      min-width: 150px;
      width: 50%;
    }
    .changeLogCommit .changedObject {
      min-width: 150px;
      width: 50%;
    }
    .changeLogCommit .transition {
      margin: 0;
    }
  }
</style>
