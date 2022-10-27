<template>
  <div class="of-dmp"
       :style="{height: height}">
    <div class="dmp-box">
      <div class="dmp-title-sign">旧</div>
      <div ref="oldBox"
           class="dmp-content"
           v-html="tmpOldValue"
           @scroll="onBoxScroll('oldBox', 'newBox')"></div>
    </div>
    <div class="dmp-box">
      <div class="dmp-title-sign">新</div>
      <div ref="newBox"
           class="dmp-content"
           v-html="tmpNewValue"
           @scroll="onBoxScroll('newBox', 'oldBox')"></div>
    </div>
  </div>
</template>

<script>
import DiffMatchPatch from 'diff-match-patch'
import './index.scss'

export default {
  name: 'DiffMatch',
  props: {
    oldValue: {
      type: String,
      default: ''
    },
    newValue: {
      type: String,
      default: ''
    },
    height: {
      type: String,
      default: '100%'
    }
  },
  data () {
    return {
      tmpOldValue: '',
      tmpNewValue: ''
    }
  },
  watch: {
    oldValue () {
      this.diffInit()
    },
    newValue () {
      this.diffInit()
    }
  },
  mounted () {
    this.diffInit()
  },
  methods: {
    onBoxScroll (current, target) {
      const _current = this.$refs[current]
      const _target = this.$refs[target]
      if (_current && _target) {
        _target.scrollTop = _current.scrollTop
      }
    },
    diffInit () {
      this.tmpOldValue = this.oldValue
      this.tmpNewValue = this.newValue
      const dmp = new DiffMatchPatch()
      const oldValue = this.oldValue.replace(/<[^>]+>/g, '')
      const newValue = this.newValue.replace(/<[^>]+>/g, '')
      const diffs = dmp.diff_main(oldValue, newValue)
      dmp.diff_cleanupSemantic(diffs)
      diffs.map(diff => {
        const [sign, text] = diff
        switch (sign) {
          case -1:
            this.tmpOldValue = this.tmpOldValue.replace(this.getDiffLine(text, this.oldValue), `<del>${text}</del>`)
            break
          case 1:
            this.tmpNewValue = this.tmpNewValue.replace(this.getDiffLine(text, this.newValue), `<ins>${text}</ins>`)
            break
        }
      })
      this.$nextTick(() => {
        this.reSetStyle()
      })
    },
    getDiffLine (text, value) {
      if (text && value) {
        const [...txtArr] = text.trim()
        const [...valArr] = value
        let labelMode = false
        let txtSign = 0
        let line = ''
        for (let i in valArr) {
          if (txtSign < txtArr.length) {
            if (valArr[i] === '<') {
              labelMode = true
            }
            if (!labelMode) {
              if (valArr[i] === txtArr[txtSign]) {
                line += valArr[i]
                txtSign++
              } else {
                line = ''
                txtSign = 0
              }
            } else if (txtSign > 0) {
              line += valArr[i]
            }
            if (valArr[i] === '>') {
              labelMode = false
            }
          } else {
            return line
          }
        }
        return line
      }
    },
    reSetStyle () {
      const oldP = this.$refs.oldBox.querySelectorAll('p')
      const newP = this.$refs.newBox.querySelectorAll('p')
      for (let i in oldP) {
        for (let j in newP) {
          if (typeof oldP[i] === 'object' && typeof newP[j] === 'object' && oldP[i].innerHTML === newP[j].innerHTML) {
            const mar = oldP[i].offsetTop - newP[j].offsetTop
            const curOMT = parseInt(getComputedStyle(oldP[i], null).marginTop)
            const curNMT = parseInt(getComputedStyle(newP[j], null).marginTop)
            if (mar > 0) {
              newP[j].style.marginTop = curNMT + mar + 'px'
            } else {
              oldP[i].style.marginTop = curOMT - mar + 'px'
            }
          }
        }
      }

      const oldImg = this.$refs.oldBox.querySelectorAll('img')
      const newImg = this.$refs.newBox.querySelectorAll('img')

      for (let i in oldImg) {
        let oldImgSrc = ''
        if (typeof oldImg[i] === 'object') {
          if (oldImg[i].className.indexOf('dmp-img-remove') < 0 && oldImg[i].className.indexOf('dmp-img-identical') < 0) {
            oldImg[i].className += ' dmp-img-remove'
          }
          oldImgSrc = oldImg[i].src
        }
        for (let j in newImg) {
          if (typeof newImg[j] === 'object') {
            if (newImg[j].className.indexOf('dmp-img-add') < 0 && newImg[j].className.indexOf('dmp-img-identical') < 0) {
              newImg[j].className += ' dmp-img-add'
            }
            if (newImg[j].src === oldImgSrc) {
              oldImg[i].className = oldImg[i].className.replace(new RegExp('dmp-img-remove', 'g'), 'dmp-img-identical')
              newImg[j].className = newImg[j].className.replace(new RegExp('dmp-img-add', 'g'), 'dmp-img-identical')
            }
          }
        }
      }
    }
  }
}
</script>
