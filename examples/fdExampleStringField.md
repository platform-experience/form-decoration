# `fdExampleStringField`

#### Template
```html
<div class="fd-ex-string"
     ng-class="{'fd-ex-mandatory': field.mandatory}"
     ng-show="field.visible">
  <label class="fd-ex-label"
         for="{{::field.name}}">{{::field.label}}</label>
  <input type="text"
         name="{{::field.name}}"
         ng-model="field.stagedValue"
         ng-focus="focus()"
         ng-blur="blur()" />
</div>
```
#### Controller function
```javascript
function fdExampleStringFieldCtrl ($scope, $timeout, $element) {
	
	$scope.focus = function () {
		$scope.hasFocus = true;
		$scope.$emit("sp.spFormField.focus", $element, $scope.inputField);
	};
	
	$scope.blur = function () {
		$scope.hasFocus = false;
		$scope.fieldValue($scope.field.stagedValue);
		$scope.$emit("sp.spFormField.blur", $element, $scope.inputField);
	};
	
	$timeout(function () {
		$scope.$emit("sp.spFormField.rendered", $element, $scope.inputField);
	});

}
```
#### CSS
```css
.fd-ex-string input {
  box-sizing: border-box;
  border: 4px solid #C4D0D6;
  border-radius: 4px;
  background-color: #F9FDFF;
  padding: 5px 15px;
  width: 100%;
}
```
