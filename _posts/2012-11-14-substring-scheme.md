---
layout: post
title:  Substrings in Scheme
tags:   snippet code
---

{% highlight scheme %}
(define substring?
  (lambda (pat str)
      (sublist? (string->list pat) (string->list str))))

(define sublist?
  (lambda (pat ls)
      (define (sublist-check pat ls pat-length ls-length)
        (cond
          [(> pat-length ls-length) #f]
          [(equal? pat (list-head ls pat-length)) #t]
          [else (sublist-check pat (cdr ls) pat-length (sub1 ls-length))]))
    (sublist-check pat ls (length pat) (length ls))))
{% endhighlight %}
