# recaptcha-angular-directive
An easy use angular directive

```js

import { Directive, ElementRef, EventEmitter, forwardRef, Input, OnInit, Output } from '@angular/core';
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';

declare const grecaptcha: any;

declare global {
  interface Window {
    grecaptcha: any;
    reCaptchaLoad: () => void
  }
}

export interface ReCaptchaConfig {
  theme?: 'dark' | 'light';
  type?: 'audio' | 'image';
  size?: 'compact' | 'normal';
  tabindex?: number;
}

@Directive({
  selector: '[Recaptcha]',
  exportAs: 'Recaptcha',
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => ReCaptchaDirective),
      multi: true
    }
  ]
})
export class ReCaptchaDirective implements OnInit, ControlValueAccessor {
  @Input() key: string;
  @Input() config: ReCaptchaConfig = {};

  @Output() captchaResponse = new EventEmitter<string>();
  @Output() captchaExpired = new EventEmitter();

  private widgetId: number;
  private render( element: HTMLElement, config ): number {
    return grecaptcha.render(element, config);
  }
  private onChange: (value: string) => void;
  private onTouched: (value:string) => void;

  constructor(
    private element: ElementRef
    ) {
  }

  ngOnInit() {
    this.registerReCaptchaCallback();
    this.addScript();
  }

  registerReCaptchaCallback() {
    window.reCaptchaLoad = () => {
      const config = {
        ...this.config,
        'sitekey': this.key,
        'callback': this.onSuccess.bind(this),
        'expired-callback': this.onExpired.bind(this)
      };
      this.widgetId = this.render(this.element.nativeElement, config);
    };
  }

  onExpired() {
    this.captchaExpired.emit();
    this.onChange(null);
    this.onTouched(null);
  }

  onSuccess(token: string) {
    this.captchaResponse.next(token);
    this.onChange(token);
    this.onTouched(token);
  }

  getId() {
    return this.widgetId;
  }

  reset(): void {
    if( !this.widgetId ) return;
    grecaptcha.reset(this.widgetId);
    this.onChange(null);
  }

  getResponse(): string {
    if( !this.widgetId )
      return grecaptcha.getResponse(this.widgetId);
  }

  addScript() {
    let script = document.createElement('script');
    script.src = 'https://www.google.com/recaptcha/api.js?onload=reCaptchaLoad&render=explicit';
    script.async = true;
    script.defer = true;
    document.body.appendChild(script);
  }

  writeValue(obj: any): void {
  }

  registerOnChange(fn: any): void {
    this.onChange = fn;
  }

  registerOnTouched(fn:any): void {
    this.onTouched = fn;
  }
}
	
```

implementation:
```html
<div Recaptcha key="yourPublicKey" (captchaResponse)="captchaResponse($event)" (captchaExpired)="captchaExpired()"></div>
```
