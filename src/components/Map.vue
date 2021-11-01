<template>
  <div class="map" @click="openUser($event)">
    <h3>Карта офиса</h3>

    <div v-if="isLoading">Loading...</div>
    <!-- Использование здесь v-if, как в воркшопе, может привести к тому,
    что DOM этой части не сгенерируется, если isLoading установлен в true.
    Этот DOM нам нужен, так как содержит все опорные компоненты.
    На воркшопе нам повезло, что на момент монтирования, isLoading мы устанавливали
    в false, дом генерировался, а после, когда в mounted мы устанавливали его в true,
    Vue еще не успевал перегенерировать DOM и мы имели доступ ко всем шаблонам.
    Однако это слишком тонкое поведение, которое слишком легко ломается,
    поэтому переписываем его на v-show-->
    <div v-show="!isLoading" class="map-root">
      <!-- v-once для оптимизации, плюс дополнительная гарантия, что эти компоненты
      не перерисуются vue (выбранная здесь архитектура решения подразумевает, что внешним
      видом этих компонентов мы управляем исключительно сами через DOM, при помощи d3)-->
      <MapSVG ref="map" v-once />
      <Table v-show="false" ref="table" v-once />
    </div>
  </div>
</template>

<script>
import MapSVG from "@/assets/images/map.svg";
import Table from "@/assets/images/workPlace.svg";
import tables from "@/assets/data/tables.json";
import legend from "@/assets/data/legend.json";
import people from "@/assets/data/people.json";
import * as d3 from "d3";

export default {
  components: {
    MapSVG,
    Table,
  },
  data() {
    return {
      //   MapSVG с самого начала необходимо достроить, поэтому сразу отключаем его отображение
      isLoading: true,
      tables: [],
      legend: [],
      //   На воркшопе здесь размещали svg, g, tableSVG но я считаю, что эти данные не участвуют в системе
      //   реактивности, поэтому в data их размещать неуместно.
    };
  },
  created() {
    this.tables = tables;
    this.legend = legend;
  },
  mounted() {
    //   Примечание из документации:
    //   "Note that mounted does not guarantee that all child components have also been mounted.
    //   If you want to wait until the entire view has been rendered,
    //   you can use vm.$nextTick inside of mounted"
    //   Мы работаем с DOM дочерних компонентов MapSVG и Table, поэтому последуем этой рекомендации
    this.$nextTick(() => {
      // Получаем шаблонные d3 компоненты здесь, вместо data.
      // Так они не будут влиять на систему реактивности
      const map = d3.select(this.$refs.map).select("g");
      //   Проверка на пустоту в d3 (проверка if(g) или if(!g), как мы делали на воркшопе,
      //   не сработает, т.к. g - это объект и он всегда true)
      if (map.empty()) {
        throw "SVG is incorrect";
      }
      this.svgTablesGroupPlace = map.append("g").classed("groupPlaces", true);
      this.svgTable = d3.select(this.$refs.table);

      this.drawTables();

      // При изменении tables или legend карту придется перерисовывать "руками".
      // Для работы drawTables нам необходим полный DOM для MapSVG и Table,
      // поэтому watch'ер назначаем только здесь, а не декларативно
      this.$watch("tables", this.drawTables, {
        deep: true,
      });
      this.$watch("legend", this.drawTables, {
        deep: true,
      });

      this.isLoading = false;
    });
  },
  methods: {
    drawTables() {
      this.svgTablesGroupPlace.html("");

      // forEach здесь уместнее, чем map
      this.tables.forEach((table) => {
        this.svgTablesGroupPlace
          .append("g")
          .attr(
            "transform",
            `translate(${table.x}, ${table.y})
              scale(0.5)
              rotate(${table.rotate || 0})`
          )
          .attr("id", table._id)
          .attr("group_id", table.group_id)
          .attr(
            "fill",
            legend.find((it) => it.group_id === table.group_id)?.color ??
              "transparent"
          )
          .classed("employer-place", true)
          .html(this.svgTable.html());
      });
    },
    openUser(event) {
      let table = event.target.closest(".employer-place");

      if (table && this.$refs.map.contains(table)) {
        this.$emit(
          "openUserByTable",
          people.find((it) => +table.id === it.tableId) ?? null
        );
      } else {
        this.$emit("closeUser");
      }
    },
  },
  emits: ["openUserByTable", "closeUser"],
};
</script>

<style scoped>
.map {
  height: 100%;
  width: 100%;
  padding: 24px;
  overflow: hidden;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
}

.map-root {
  height: 100%;
  width: 100%;
  overflow: hidden;
  box-sizing: border-box;
}

h3 {
  margin-top: 0px;
}

::v-deep svg {
  height: 100%;
  width: 100%;
}

::v-deep .table {
  cursor: pointer;
}
</style>
