import Vue from 'vue';
Vue.directive('paroller', {
	bind: function(el, binding) {
		let working = false;
		let scrollAction = function() {
			working = false;
		};

		// Methods
		let getScroll = {
			top: function(elem) {
				let rect = elem.getBoundingClientRect();
				let top = rect.top;
				if (isNaN(top)) {
					top = 0;
				}
				return top;
			},
			left: function(elem) {
				let rect = elem.getBoundingClientRect();
				return rect.left;
			},
			bottom: function(elem) {
				let rect = elem.getBoundingClientRect();
				return rect.top + rect.height;
			},
			height: function(elem) {
				let rect = elem.getBoundingClientRect();
				return rect.height;
			},
			windowTop: function() {
				let top = (window.pageYOffset || document.scrollTop) - (document.clientTop || 0);
				if (isNaN(top)) {
					top = 0;
				}
				return top;
			}
		};

		let setDirection = {
			bgVertical: function(elem, bgOffset) {
				elem.style.backgroundPosition = 'center ' + -bgOffset + 'px';
			},
			bgHorizontal: function(elem, bgOffset) {
				elem.style.backgroundPosition = -bgOffset + 'px' + ' center';
			},
			vertical: function(elem, elemOffset, oldTransform) {
				oldTransform === 'none' ? (oldTransform = '') : true;
				elem.style.transform = 'translateY(' + elemOffset + 'px)' + oldTransform;
				elem.style.transition = 'transform linear';
				elem.style.willChange = 'transform';
			},
			horizontal: function(elem, elemOffset, oldTransform) {
				oldTransform === 'none' ? (oldTransform = '') : true;
				elem.style.transform = 'translateX(' + elemOffset + 'px)' + oldTransform;
				elem.style.transition = 'transform linear';
				elem.style.willChange = 'transform';
			}
		};

		let setMovement = {
			factor: function(elem, width, options) {
				let dataFactor = binding.value.factor || 0;
				let factor = dataFactor ? dataFactor : options.factor;
				if (width < 576) {
					let dataFactorXs = binding.value.factorXs || 0;
					let factorXs = dataFactorXs ? dataFactorXs : options.factorXs;
					return factorXs ? factorXs : factor;
				} else if (width <= 768) {
					let dataFactorSm = binding.value.factorSm || 0;
					let factorSm = dataFactorSm ? dataFactorSm : options.factorSm;
					return factorSm ? factorSm : factor;
				} else if (width <= 1024) {
					let dataFactorMd = binding.value.factorMd || 0;
					let factorMd = dataFactorMd ? dataFactorMd : options.factorMd;
					return factorMd ? factorMd : factor;
				} else if (width <= 1200) {
					let dataFactorLg = binding.value.factorLg || 0;
					let factorLg = dataFactorLg ? dataFactorLg : options.factorLg;
					return factorLg ? factorLg : factor;
				} else if (width <= 1920) {
					let dataFactorXl = binding.value.factorXl || 0;
					let factorXl = dataFactorXl ? dataFactorXl : options.factorXl;
					return factorXl ? factorXl : factor;
				} else {
					return factor;
				}
			},
			bgOffset: function(offset, factor) {
				return Math.round(offset * factor);
			},
			transform: function(offset, factor, windowHeight, height) {
				return Math.round((offset - windowHeight / 2 + height) * factor);
			}
		};

		let clearPositions = {
			background: function(elem) {
				elem.style.backgroundPosition = 'unset';
			},
			foreground: function(elem) {
				elem.style.transform = 'unset';
				elem.style.transition = 'unset';
			}
		};

		// The working

		let windowHeight = window.outerHeight;
		let documentHeight = document.outerHeight;

		let options = {
			factor: 0, // - to +
			factorXs: 0, // - to +
			factorSm: 0, // - to +
			factorMd: 0, // - to +
			factorLg: 0, // - to +
			factorXl: 0, // - to +
			type: 'background', // foreground
			direction: 'vertical' // horizontal
		};

		let width = window.width;
		let offset = el.offsetTop;
		let height = getScroll.height(el);
		let dataType = binding.value.type;
		let dataDirection = binding.value.direction;
		let oldStyle = getComputedStyle(el, null);
		let oldTransform = oldStyle.getPropertyValue('transform');
		let type = dataType || options.type;
		let direction = dataDirection || options.direction;
		let factor = setMovement.factor(el, width, options);
		let bgOffset = setMovement.bgOffset(offset, factor);
		let transform = setMovement.transform(offset, factor, windowHeight, height);

		if (type === 'background') {
			if (direction === 'vertical') {
				setDirection.bgVertical(el, bgOffset);
			} else if (direction === 'horizontal') {
				setDirection.bgHorizontal(el, bgOffset);
			}
		} else if (type === 'foreground') {
			if (direction === 'vertical') {
				setDirection.vertical(el, transform, oldTransform);
			} else if (direction === 'horizontal') {
				setDirection.horizontal(el, transform, oldTransform);
			}
		}

		window.addEventListener('resize', function() {
			let scrolling = getScroll.top(el);
			width = window.width;
			offset = el.offsetTop;
			height = el.outerHeight;
			factor = setMovement.factor(el, width, options);

			bgOffset = Math.round(offset * factor);
			transform = Math.round((offset - windowHeight / 2 + height) * factor);

			if (!working) {
				window.requestAnimationFrame(scrollAction);
				working = true;
			}

			if (type === 'background') {
				clearPositions.background(el);
				if (direction === 'vertical') {
					setDirection.bgVertical(el, bgOffset);
				} else if (direction === 'horizontal') {
					setDirection.bgHorizontal(el, bgOffset);
				}
			} else if (type === 'foreground' && scrolling <= documentHeight) {
				clearPositions.foreground(el);
				if (direction === 'vertical') {
					setDirection.vertical(el, transform);
				} else if (direction === 'horizontal') {
					setDirection.horizontal(el, transform);
				}
			}
		});
		window.addEventListener('scroll', function() {
			let scrolling = getScroll.top(el);
			documentHeight = document.height;

			bgOffset = Math.round((offset - scrolling) * factor);
			transform = Math.round((offset - windowHeight / 2 + height - scrolling) * factor);
			if (!working) {
				window.requestAnimationFrame(scrollAction);
				working = true;
			}
			if (type === 'background' && getScroll.windowTop() <= getScroll.bottom(el) + 100) {
				if (direction === 'vertical') {
					setDirection.bgVertical(el, bgOffset);
				} else if (direction === 'horizontal') {
					setDirection.bgHorizontal(el, bgOffset);
				}
			} else if (type === 'foreground') {
				if (direction === 'vertical') {
					setDirection.vertical(el, transform, oldTransform);
				} else if (direction === 'horizontal') {
					setDirection.horizontal(el, transform, oldTransform);
				}
			}
		});
	}
});