# init.el for emacs

```lisp
;; init.el for emacs
;; Copyright (C) 2020 iotang
;;  
;; This program is free software: you can redistribute it and/or modify
;; it under the terms of the GNU General Public License as published by
;; the Free Software Foundation, either version 3 of the License, or
;; (at your option) any later version.
;;  
;; This program is distributed in the hope that it will be useful,
;; but WITHOUT ANY WARRANTY; without even the implied warranty of
;; MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
;; GNU General Public License for more details.
;;  
;; You should have received a copy of the GNU General Public License
;; along with this program.  If not, see <https://www.gnu.org/licenses/>.


;; 要想使这个配置起效，请把它命名为 `.emacs'，然后放到你的账户的用户目录下 (/home/<账户名字>/.emacs)；
;; 或者是在账户的用户目录建立目录 `.emacs.d'(/home/<账户名字>/.emacs.d/)，然后把这个配置命名为 `init.el'，放到 `.emacs.d' 下。


;; t = True
;; nil = False




;; === 基本配置 === ;;

;; 默认编码环境为 UTF-8。
(set-language-environment "UTF-8")
(set-default-coding-systems 'utf-8)

;; 不显示欢迎页面。
(setq-default inhibit-startup-screen t)

;; emacs 和系统的剪贴板共用。
(setq-default x-select-enable-clipboard t)

;; 设置标题。
(setq-default frame-title-format "%b - emacs")

;; 全选大文件的时候不会卡死。当然，如果你使用的是 fcitx，并且打开了剪贴板支持，在复制大文件的时候还是会卡死。你可以使用 emacs-terminal 来解决这个问题。
(setq-default select-active-regions 'only)

;; emacs-terminal 鼠标可用。
(xterm-mouse-mode t)

;; 鼠标滚轮支持。
(mouse-wheel-mode t)

;; 启用 cua-mode （C-x C-c C-v 分别代表剪切，复制，粘贴）。
(cua-mode t)

;; 回答 yes/no 改成回答 y/n。
(fset 'yes-or-no-p 'y-or-n-p)




;; === 观感 === ;;

;; 不显示工具栏。
(tool-bar-mode nil)

;; 显示行号。
(global-linum-mode t)

;; 显示列号。
(setq column-number-mode t)

;; 在 buffer 分界条上显示（24小时制）时间。
(setq-default display-time-24hr-format t)
(setq-default display-time-interval 1)
(display-time-mode 1)

;; 高亮当前行。
(global-hl-line-mode 1)

;; 高亮匹配括号。
(show-paren-mode t)
(setq show-paren-style 'parenthesis)

;; 光标形状。'box = 框；'bar = 竖线。
(setq-default cursor-type 'bar)

;; 优化页面滚动。
(setq-default scroll-step 1 scroll-margin 3 scroll-conservatively 10000)

;; 透明度。
(set-frame-parameter (selected-frame) 'alpha (list 90 70))
(add-to-list 'default-frame-alist (cons 'alpha (list 90 70)))




;; === 快捷键 === ;;

;; Ctrl-z 撤销。
(global-set-key (kbd "C-z") 'undo)

;; Ctrl-a 全选。
(global-set-key (kbd "C-a") 'mark-whole-buffer)

;; Ctrl-h 替换。
(define-key key-translation-map (kbd "C-h") (kbd "M-%"))

;; Ctrl-s 保存。
(global-set-key (kbd "C-s") 'save-buffer)

;; 其它快捷键。
(setq cua-mode t)
(global-set-key (kbd "C-<tab>") 'other-window)
(global-set-key (kbd "C-w") 'delete-window)
(global-set-key (kbd "C-f") 'isearch-forward)
(define-key key-translation-map (kbd "M-<up>") (kbd "C-x <left>"))
(define-key key-translation-map (kbd "M-<down>") (kbd "C-x <right>"))
(define-key key-translation-map (kbd "M-<left>") (kbd "C-x <left>"))
(define-key key-translation-map (kbd "M-<right>") (kbd "C-x <right>"))




;; === 编辑器 === ;;

;; 撤销记录扩大。
(setq-default kill-ring-max 65535)

;; 开启备份文件。（~/.emacs.d/auto-save-list）
(setq-default auto-save-mode t)
(setq-default backup-by-copying t)

;; 设置备份文件时间间隔。这个单位是秒。
(setq-default auto-save-timeout 30)

;; 自动重载文件。
(global-auto-revert-mode t)

;; 优化文件树结构。
(ido-mode t)

;; 语法高亮。
(global-font-lock-mode t)




;; === C++ IDE === ;;

;; C++ 代码风格。
;; "bsd" = 所有大括号换行。这是真理
;; "java" = 所有大括号不换行。else 接在右大括号后面。
;; "k&r" = "awk" = 只有命名空间旁、定义类、定义函数时的大括号换行。else 接在右大括号后面。
;; "stroustrup" = 只有命名空间旁、定义类、定义函数时的大括号换行。else 换行。
;; "whitesmith" = 所有大括号换行。大括号有一次额外缩进。
;; "gnu" = 所有大括号换行。每次左括号开始，会有一层额外缩进。这是 emacs 默认
;; "linux" = 只有命名空间旁、定义类、定义函数时的大括号换行。else 接在右大括号后面。一般来说，这个风格应该有 8 格的空格缩进。
(setq-default c-default-style "bsd")

;; C++ 代码缩进单位长度。
(setq-default c-basic-offset 3)

;; 使用 tab 缩进。
(setq-default indent-tabs-mode t)

;; tab 的长度。务必和缩进长度一致。
(setq-default default-tab-width 3)
(setq-default tab-width 3)

;; 换行的时候自动缩进。
(global-set-key (kbd "RET") 'newline-and-indent)

;; 一键编译。
(defun compile-file ()(interactive)(compile (format "g++ %s -o %s -g -lm -Wall -std=c++98 -fsanitize=address" (buffer-name)(file-name-sans-extension (buffer-name)))))
;; (defun compile-file ()(interactive)(compile (format "g++ %s -o %s -g -lm -Wall" (buffer-name)(file-name-sans-extension (buffer-name)))))
(global-set-key [f9] 'compile-file)
(global-set-key [f10] 'compile)

;; 一键 GDB。
(global-set-key [f2] 'gud-gdb)




;; === 配色方案 === ;;

;; yyb 配色方案，来自<https://www.cnblogs.com/cjyyb/p/8366042.html>。
(setq default-frame-alist
		'((vertical-scroll-bars)
		  (width . 80)
		  (height . 40)
		  (tool-bar-lines . 0)
		  (menu-bar-lines . 1)
		  (scroll-bar-lines . 0)
		  (right-fringe)
		  (left-fringe)))
(set-background-color "grey20")
(set-foreground-color "grey80")
(set-cursor-color "gold")
(set-mouse-color "gold")
(set-face-background 'highlight "grey10")
;; (set-face-foreground 'region "cyan")
(set-face-background 'region "grey35")
;; (set-face-foreground 'secondary-selection "skyblue")
(set-face-background 'secondary-selection "darkblue")

(custom-set-faces
 ;; custom-set-faces was added by Custom.
 ;; If you edit it by hand, you could mess it up, so be careful.
 ;; Your init file should contain only one such instance.
 ;; If there is more than one, they won't work right.
 '(default ((t (:family "Monospace" :foundry "unknown" :slant normal :weight normal :height 98 :width normal)))))
```