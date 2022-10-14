<template>
  <section class="chat-box">
    <div class="chat-header">현대백화점 챗봇 데모</div>
    <div class="chat-box-list-container" ref="chatbox">
      <ul class="chat-box-list">
        <li
          class="message"
          v-for="(message, idx) in messages"
          :key="idx"
          :class="message.author"
        >
          <div>
            <!-- <p> -->
            <div v-html="message.text"></div>
            <!-- </p> -->
            <img
              v-if="message.image"
              :src="require(`~/assets/img/${message.image}`)"
              alt="info-image"
              class="info-image"
            />
          </div>
        </li>
      </ul>
    </div>
    <div class="chat-inputs">
      <input type="text" v-model="message" @keyup.enter="sendMessage" />
      <button @click="sendMessage">입력</button>
    </div>
  </section>
</template>

<script>
export default {
  name: "ChatBox",
  data: () => ({
    message: "",
    messages: [
      {
        text: "안녕하세요. 현대백화점 챗봇(데모)입니다. \
        <br/>저는 아직 5가지 말밖에 이해하지 못 해요. \
        <br/>그리고 현대백화점 압구정 본점 데이터만 가지고 있어요. \
        <br/><li class='chat-li'>\
        <ul>👉 운영 안내</ul>\
        <ul>👉 마일리지 혜택</ul>\
        <ul>👉 카드 혜택</ul>\
        <ul>👉 주차비 안내</ul>\
        <ul>👉 위치 안내</ul>\
        </li>",
        image: null,
        author: "server",
      },
    ],
  }),
  methods: {
    sendMessage() {
      const message = this.message;

      this.messages.push({
        text: message,
        author: "client",
      });

      this.message = "";

      this.$axios
        .post(`https://chat-hyun.herokuapp.com/chat`, {
          question: message,
        })
        .then((res) => {
          const intent = res.data.intent;
          let text = "";
          let imgName = null;

          if (intent == "ask_operation") {
            imgName = "holiday.png";
            text =
              "<span><p>현대백화점 압구정 본점 안내해드리겠습니다</p></span>";
          } else if (intent == "ask_milage_reward") {
            imgName = "milage_reward.png";
            text =
              "<span><p>현대백화점 마일리지 혜택 안내해드리겠습니다</p></span>";
          } else if (intent == "ask_card_benefit") {
            imgName = "card_benefit.png";
            text =
              "<span><p>현대백화점 카드 혜택 안내해드리겠습니다</p></span>";
          } else if (intent == "ask_parking_fee") {
            imgName = "parking.png";
            text = "<span><p>현대백화점 주차비 안내해드리겠습니다</p></span>";
          } else if (intent == "ask_place") {
            imgName = null;
            text = "";
            const response_info = res.data.info;
            if (response_info) {
              text += `<span><p>현대백화점 장소 안내해드리겠습니다</p></span>\
              <div class="info-wrapper">`;
              response_info.forEach((info_row) => {
                text += `<div class="info-row">\
                <section class="info-name">${info_row.name}</section>\
                <section>${info_row.category}</section>\
                <section>${info_row.floor}</section>`;
                if (info_row.tel != "-")
                  text += `<section>${info_row.tel}</section>`;
                text += `</div>`;
              });
              text += `</div>`;
            } else {
              text +=
                "<span><p>죄송합니다. 해당 장소를 제가 아직 잘 몰라요.</p></span>";
            }
          }

          this.messages.push({
            text: text,
            image: imgName,
            author: "server",
          });

          this.$nextTick(() => {
            this.$refs.chatbox.scrollTop = this.$refs.chatbox.scrollHeight;
          });
        });
    },
  },
};
</script>

<style lang="scss">
.info-image {
  width: 300px;
  @media screen and (max-width: 600px) {
    width: 50vw;
  }
  object-fit: contain;
}

.info-wrapper {
  background-color: rgb(236, 236, 236);
  border: 1px solid black;
}

.info-row {
  border-bottom: 1px solid black;
  section {
    font-size: 0.5rem;
  }
  section.info-name {
    font-size: 1rem;
  }
}

.chat-header {
  text-align: center;
  background-color: #999;
  padding: 1rem;
  color: white;
}

.chat-box,
.chat-box-list {
  display: flex;
  flex-direction: column;
  list-style-type: none;
}

.chat-box-list-container {
  overflow: scroll;
  margin-bottom: 1px;
}

.chat-box-list {
  padding-left: 10px;
  padding-right: 10px;

  .client {
    color: white;
  }

  div {
    padding: 8px;
    border-radius: 10px;
  }

  .server {
    // span {
    //   }
    > div {
      background: #99cc00;
      float: left;
    }
  }

  .client {
    > div {
      background: #0070c8;
      float: right;
    }
  }
}

.chat-box-list > li {
  margin-bottom: 2rem;
}

.chat-box {
  // margin: 10px;
  border: 1px solid #999;
  width: 50vw;
  @media screen and (max-width: 600px) {
    width: 95vw;
  }
  height: 100vh;
  border-radius: 4px;
  margin-left: auto;
  margin-right: auto;
  align-items: space-between;
  justify-content: space-between;
}

.chat-box-list {
  height: 90vh;
}

.chat-inputs {
  display: flex;
  height: 8vh;

  input {
    line-height: 3;
    width: 100%;
    border: 1px solid #999;
    border-left: none;
    border-bottom: none;
    border-right: none;
    border-bottom-left-radius: 4px;
    padding-left: 15px;
  }

  button {
    width: 145px;
    color: white;
    background: #0070c8;
    border-color: #999;
    border-bottom: none;
    border-right: none;
    border-bottom-right-radius: 3px;
  }
}
</style>
