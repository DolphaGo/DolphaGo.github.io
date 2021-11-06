---
layout: post
title:  "Hello World!"
date:   2021-11-11 05:18:00
categories: Say
---

Test...

Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll's GitHub repo][jekyll-gh].

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com

```java
@PostMapping("/add")
public String addItemV3(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

    log.info("objectName={}", bindingResult.getObjectName()); // objectName=item
    log.info("target={}", bindingResult.getTarget()); // target=Item(id=null, itemName=, price=10, quantity=2)

    // 검증 로직
    if (!StringUtils.hasText(item.getItemName())) {
        bindingResult.addError(new FieldError("item", "itemName", item.getItemName(), false, new String[] { "required.item.itemName" }, null, "상품 이름은 필수입니다."));
    }

    if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
        bindingResult.addError(
                new FieldError("item", "price", item.getPrice(), false, new String[] { "range.item.price" }, new Object[] { 1000, 1000000 }, "가격은 1,000 ~ 1,000,000 까지만 허용합니다."));
    }

    if (item.getQuantity() == null || item.getQuantity() > 9999 || item.getQuantity() <= 0) {
        bindingResult.addError(
                new FieldError("item", "quantity", item.getQuantity(), false, new String[] { "max.item.quantity" }, new Object[] { 9999 }, "수량은 1개이상, 최대 9,999까지 허용합니다."));
    }

    // 특정 필드가 아닌 복합 룰 검증
    if (item.getPrice() != null && item.getQuantity() != null) {
        final int resultPrice = item.getPrice() * item.getQuantity();
        if (resultPrice < 10000) {
            bindingResult.addError(
                    new ObjectError("item", new String[] { "totalPriceMin", "required.default" }, new Object[] { 10000, resultPrice },
                                    "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
            // new String[]인 이유는, 순서대로 우선순위를 갖고 있기 때문입니다.
            // default message 조차 없으면 오류가 나게 됩니다.
        }
    }

    // 검증에 실패하면 다시 입력 폼으로
    if (bindingResult.hasErrors()) {
        log.info("errors = {}", bindingResult);
        // bindingResult는 자동으로 view에 담아갑니다. 그래서 따로 modelAttribute로 담지 않아도 됩니다.
        // 스프링에서 BindingResult가 기본적으로 되어있다는 것이 그런 것들을 포함하고 있는 의미이기 때문입니다.
        return "validation/v2/addForm";
    }

    // 성공 로직
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/validation/v2/items/{itemId}";
}
```

이미지 테스트
![](../assets/images/2021-11-11-03-45-06.png)