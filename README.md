black-litterman
===============
data=xlsread('data.xlsx');
us_e=data(:,1);
glo_e=data(:,2);
bond=data(:,3);

date=[];
for n=1:length(data)
    date(n,1)=1990+floor(n/12)+rem(n,12)/12;
end

%cumulative returns plot
ret_us=cumprod(us_e+ones(length(us_e),1));
ret_glo=cumprod(glo_e+ones(length(glo_e),1));
ret_bond=cumprod(bond+ones(length(bond),1));
% plot(date,ret_us,date,ret_glo,date,ret_bond)
% legend('us_e','glo_e','bond')
% title('cumulative returns 1990-2012')

%we see during 2004 and 2007 equity is doing really well
us_sample=us_e(floor(date)>=2004&floor(date)<=2007);
glo_sample=glo_e(floor(date)>=2004&floor(date)<=2007);
bond_sample=bond(floor(date)>=2004&floor(date)<=2007);
date_sample=date(floor(date)>=2004&floor(date)<=2007);

%taking geometric mean of return 2004-2007
ret_us_sam=cumprod(us_sample+ones(length(us_sample),1));
r_us=(ret_us_sam(end))^(1/4)-1;
ret_glo_sam=cumprod(glo_sample+ones(length(glo_sample),1));
r_glo=(ret_glo_sam(end))^(1/4)-1;
ret_bond_sam=cumprod(bond_sample+ones(length(bond_sample),1));
r_bond=(ret_bond_sam(end))^(1/4)-1;

%mean variance to get weight
lambda=4;
cov_sam=cov([us_sample glo_sample bond_sample]);
r_sam=[r_us;r_glo;r_bond];
w_sam=inv(cov_sam)*r_sam/lambda;
w_sam=w_sam/sum(w_sam);
% bar(w_sam')
% labels={'us equity','global_equity','bond'};
% set(gca,'XTickLabel',labels)

%back test
ret_sam=ret_us_sam*w_sam(1)+ret_glo_sam*w_sam(2)+ret_bond_sam*w_sam(3);
% plot(date_sample,ret_us_sam,date_sample,ret_glo_sam,date_sample,ret_bond_sam,date_sample,ret_sam);
% legend('us_equity','global_equity','bond','portfolio')
% title('cumulative return 2004-2007')

%test out of sample 2008-2010
us_sample2=us_e(floor(date)>=2008&floor(date)<=2010);
glo_sample2=glo_e(floor(date)>=2008&floor(date)<=2010);
bond_sample2=bond(floor(date)>=2008&floor(date)<=2010);
date_sample2=date(floor(date)>=2008&floor(date)<=2010);

ret_us_sam2=cumprod(us_sample2+ones(length(us_sample2),1));
ret_glo_sam2=cumprod(glo_sample2+ones(length(glo_sample2),1));
ret_bond_sam2=cumprod(bond_sample2+ones(length(bond_sample2),1));
ret_sam2=ret_us_sam2*w_sam(1)+ret_glo_sam2*w_sam(2)+ret_bond_sam2*w_sam(3);
% plot(date_sample2,ret_us_sam2,date_sample2,ret_glo_sam2,date_sample2,ret_bond_sam2,date_sample2,ret_sam2);
% legend('us_equity','global_equity','bond','portfolio')
% title('cumulative return 2008-2010')

%black litterman method
w_us=0.4;
w_glo=0.2;
w_bond=0.4;
w_eq=[w_us;w_glo;w_bond];

r_implied=lambda*cov_sam*w_eq;     %reverse optimization
tau=1/4;
p1=[-1 0 1];                       %bond same return as us eq
p2=[1 0 0];                        %us equity return 3%
p=[p1;p2];
omega1=0.01^2;
omega2=0.03^2;
omega=[omega1 0;0 omega2];
q=[0;0.03];

cov_BL=inv(inv(tau*cov_sam)+p'*inv(omega)*p);
ret_BL=cov_BL*(inv(tau*cov_sam)*r_implied+p'*inv(omega)*q);
cov_pred=cov_BL+cov_sam;
w_BL=inv(cov_pred)*(ret_BL./lambda);
w3_1=w_BL(1)/sum(w_BL);
w3_2=w_BL(2)/sum(w_BL);
w3_3=w_BL(3)/sum(w_BL);
w_BL=[w3_1;w3_2;w3_3];

%test out sample 2008-2010
ret_sam3=ret_us_sam2*w_BL(1)+ret_glo_sam2*w_BL(2)+ret_bond_sam2*w_BL(3);
plot(date_sample2,ret_us_sam2,date_sample2,ret_glo_sam2,date_sample2,ret_bond_sam2,date_sample2,ret_sam3);
legend('us_equity','global_equity','bond','portfolio')
title('cumulative return 2008-2010')

%weight comparison
w_grouped=[w_sam w_BL];
 bar(w_grouped,'grouped')
 labels={'us_e','global_e','bond'};
 set(gca,'XTickLabel',labels)
 legend('mean_variance','black_litterman')
 title('weights')
