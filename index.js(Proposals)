/* eslint-disable jsx-a11y/anchor-is-valid */
import React, { Component } from 'react';
import { connect } from 'react-redux';
import { bindActionCreators } from 'redux';
import get from 'lodash/get';
import Footer from 'components/Footer';
import Button from 'components/Button';
import Avatar from 'components/Avatar';
import Chat from 'components/Chat';
import EnvironmentItem from 'components/EnvironmentItem';
import Checkbox from 'components/Checkbox';
import Modal from 'components/Modal';
import Tooltip from 'components/Tooltip';
import SavingChart from 'components/SavingChart';
import Expand from 'components/Expand';
import SunroofImage from 'components/SunroofImage';
import SunroofInfo from 'components/SunroofInfo';
import { ReactComponent as CarbonDioxideIcon } from 'assets/images/icon-carbon-dioxide.svg';
import { ReactComponent as LeafIcon } from 'assets/images/icon-leaf.svg';
import { ReactComponent as StationIcon } from 'assets/images/icon-station.svg';
import { ReactComponent as TooltipIcon } from 'assets/images/icon-tooltip.svg';
import imgStop from 'assets/images/stop.png';
import { ReactComponent as GiftIcon } from 'assets/images/icon-gift.svg';
import {
  setUserData,
  saveSession,
  navigateTo,
  trackForwardProgress,
} from 'store/actions';
import { engineSolarFindBuilding } from 'common/api.mock';
import { formatNumber, checkIfSunroofIsLoaded } from 'common/utils';
import { withTranslation, Trans } from 'react-i18next';
import { getLocalStorageItem } from 'common/localStorage';
import { VARIATION_OPTIONS } from 'pages/constants';
import StyledProposals, { CircleText } from './styled';

class Proposals extends Component {
  constructor(props) {
    super(props);

    const variation = getLocalStorageItem('variation');
    this.state = {
      showDialog: false,
      checked: false,
      loading: true,
      squareFeet: null,
      sunshineHours: null,
      centerLat: null,
      centerLng: null,
      error: null,
      variation: VARIATION_OPTIONS[variation],
    };

    this.divRef = React.createRef();
  }

  componentDidMount() {
    this.initData();
  }

  initData = async () => {
    const { theme, user } = this.props;
    if (!theme.is_sunroof_enabled) {
      this.setState({ loading: false, error: 'Skip sunroof' });
      return;
    }
    const { data: building } = await engineSolarFindBuilding(
      { latitude: user.rooftop_latitude, longitude: user.rooftop_longitude },
      theme.google_api_gateway_key
    );
    if (!building) {
      this.setState({
        loading: false,
        error: 'Requested location is not supported by Google Sunroof API.',
      });
    } else {
      const squareFeet = formatNumber(
        Math.round(
          get(building, 'solarPotential.maxArrayAreaMeters2', 0) * 10.764
        )
      );
      const centerLat = get(building, 'center.latitude', 0);
      const centerLng = get(building, 'center.longitude', 0);
      const sunshineHours = formatNumber(
        Math.round(get(building, 'solarPotential.maxSunshineHoursPerYear', 0))
      );
      await checkIfSunroofIsLoaded(centerLat, centerLng);
      this.setState({
        loading: false,
        squareFeet,
        centerLat,
        centerLng,
        sunshineHours,
      });
    }
    // setTimeout(() => {
    //   this.setState({ showDialog: true });
    // }, 4000);
  };

  handleCloseDialog = () => {
    const { checked } = this.state;
    if (!checked) {
      return;
    }
    this.setState({ showDialog: false });
  };

  // handleNext = async () => {
  //   const { actions } = this.props;
  //   const { checked } = this.state;
  //   actions.setUserData({
  //     step: '/qualify/default1',
  //     terms_consent: checked ? 1 : null,
  //   });
  //   await actions.saveSession();
  //   actions.trackForwardProgress();
  //   await actions.navigateTo('/qualify/default1');
  // };

  handleCheck = (event) => {
    this.setState({ checked: event.target.checked });
  };

  render() {
    const { user, theme, t } = this.props;
    const {
      showDialog,
      checked,
      loading,
      sunshineHours,
      centerLat,
      centerLng,
      squareFeet,
      error,
      variation,
    } = this.state;
    const data = { ...user };
    const projectSunroof =
      !loading &&
      !error &&
      centerLat &&
      centerLng &&
      (squareFeet || sunshineHours);
    const { disclosure, company_name } = theme;
    const { utm_source: requestId } = user;

    return (
      <>
        <variation.header showRestartButton={false} />
        <div className="main-container">
          <div
            className="main"
            style={{
              height: !loading ? 'initial' : null,
            }}
          >
            {loading ? (
              <div className="content">
                <div className="form">
                  <div className="form-content">
                    <Chat mode="loading" loading={loading} />
                  </div>
                </div>
              </div>
            ) : (
              <StyledProposals className="content result-3-page">
                <div className="form">
                  <div className="white-section text-center">
                    <div className="section-container">
                      <div className="mb-16">
                        <Avatar size={78} />
                      </div>
                      <div className="mb-16 introduction">
                        {t('introduction', {
                          city: data.city,
                          state: data.state,
                          avg_bill: formatNumber(data.avg_bill),
                        })}
                      </div>
                      {projectSunroof && (
                        <div className="sunroof-container mt-32">
                          {centerLat && centerLng && (
                            <div className="image-container">
                              <SunroofImage lat={centerLat} lng={centerLng} />
                            </div>
                          )}
                          {(sunshineHours || squareFeet) && (
                            <SunroofInfo
                              sunshineHours={sunshineHours}
                              squareFeet={squareFeet}
                              className="mt-16"
                            />
                          )}
                        </div>
                      )}
                    </div>
                  </div>
                  <div className="dark-section text-center" ref={this.divRef}>
                    <div className="section-container">
                      <div className="section-title-variation mb-16">
                        {t(
                          'Replace or greatly reduce your utility bill with solar for just...'
                        )}
                      </div>
                      <div className="mb-16">
                        <CircleText>
                          <div className="circle-text-main">
                            {theme.price_system_display ||
                            !theme.is_premier_account ? (
                              `$${formatNumber(
                                Math.round(data.monthly_payment_min)
                              )}-${formatNumber(
                                Math.round(data.monthly_payment_max)
                              )}*`
                            ) : data.monthly_payment_best_guess ? (
                              <span className="exact-pricing-main-number">
                                $
                                {formatNumber(
                                  Math.round(data.monthly_payment_best_guess)
                                )}
                                *
                              </span>
                            ) : (
                              `$${formatNumber(
                                Math.round(data.monthly_payment_min)
                              )}-${formatNumber(
                                Math.round(data.monthly_payment_max)
                              )}*`
                            )}
                          </div>
                          <div
                            className={`circle-text-small ${
                              theme.price_system_display ||
                              !theme.is_premier_account
                                ? ''
                                : 'exact-pricing-main-number'
                            }`}
                          >
                            {t('per month')}
                          </div>
                        </CircleText>
                      </div>
                      <div className="bold-text">
                        {t(
                          '*{{product_apr}}% APR financing for {{product_term}} years on approved credit with zero down.',
                          {
                            product_apr: theme.product_apr || 3.99,
                            product_term: theme.product_term || 20,
                          }
                        )}
                      </div>
                    </div>
                  </div>
                  <div className="black-section">
                    <div className="section-container">
                      <div className="discount-block">
                        <div className="bg-circle-icon">
                          <GiftIcon />
                        </div>
                        <div className="discount-info">
                          <div className="discount-title">
                            {t('Total Incentive Discounts')}
                          </div>
                          <div className="discount-value">
                            ${`${formatNumber(data.incentive)}`}
                            <sup>**</sup>
                          </div>
                          <div className="divider">
                            <span className="divider-text">
                              {t('This year only!')}
                            </span>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                  <div className="dark-section text-center">
                    <div className="section-container">
                      <div className="section-title text-normal">
                        {t('25 Year Savings')}
                      </div>
                      <div className="section-description mb-32">
                        ${`${formatNumber(data.savings)}`}
                      </div>
                      <SavingChart
                        value1={data.avg_bill}
                        value2={formatNumber(
                          Math.round(data.avg_bill * 1.03 ** 10)
                        )}
                        value3={formatNumber(
                          Math.round(data.avg_bill * 1.03 ** 25)
                        )}
                      />
                    </div>
                  </div>
                  <div className="expand-section dark">
                    <div className="section-container">
                      <Expand title={t('Why do utility rates go up?')}>
                        {t(
                          'Utility rates in the US historically increased by 3% each year. This is based on a variety of market factors including inflation.'
                        )}
                      </Expand>
                    </div>
                  </div>
                  <div className="white-section text-center">
                    <div className="section-container">
                      <div className="section-title">
                        {t('Home Value Increase')}
                      </div>
                      <div className="section-description">4.1%</div>
                      <div className="subtext">
                        {t('According to a recent study from Zillow.*')}
                      </div>
                    </div>
                  </div>
                  <div className="expand-section">
                    <div className="section-container">
                      <Expand title={t('Learn about the Zillow study.')}>
                        {t(
                          'Zillow calculated the solar premium by comparing homes nd without solar-energy systems that were listed for sale and sold from March 1, 2018 to February 28, 2019, controlling for observable attributes of the homes.'
                        )}
                      </Expand>
                    </div>
                  </div>
                  <div className="dark-section text-center">
                    <div className="section-container">
                      <div className="section-title">
                        {t('Environmental Impact')}
                      </div>
                      <div className="environmental-list">
                        <EnvironmentItem
                          icon={<CarbonDioxideIcon />}
                          title={`${data.co2_tons}`}
                          content={t('Tons of CO2 avoided')}
                        />
                        <EnvironmentItem title="=" />
                        <EnvironmentItem
                          icon={<LeafIcon />}
                          title={`${data.tree_seedlings}`}
                          content={t('Trees grown for 10 years')}
                        />
                        <EnvironmentItem title="=" />
                        <EnvironmentItem
                          icon={<StationIcon />}
                          title={`${data.gal_of_gas}`}
                          content={t('Gallons of gas not used')}
                        />
                      </div>
                    </div>
                  </div>
                  <div className="white-section text-center">
                    <div className="section-container">
                      <div className="section-title">{t('System Size')}</div>
                      <div
                        className="section-description"
                        style={{ whiteSpace: 'nowrap' }}
                      >
                        {theme.price_system_display ||
                        !theme.is_premier_account ? (
                          `${data.install_size_min}-${data.install_size_max}`
                        ) : (
                          <span className="exact-system-size-number">
                            {data.install_size}
                          </span>
                        )}
                        <small>kW</small>{' '}
                        <TooltipIcon data-for="tooltip" data-tip />
                        <Tooltip id="tooltip" place="left">
                          <div className="tooltip-content">
                            {t('{{panels_min}}-{{panels_max}} Panels', {
                              panels_min: data.panels_min,
                              panels_max: data.panels_max,
                            })}
                          </div>
                        </Tooltip>
                      </div>
                    </div>
                  </div>
                </div>
              </StyledProposals>
            )}
            {showDialog && (
              <Modal isOpen={showDialog} className="stop-modal">
                <div className="my-32">
                  <img src={imgStop} alt="stop" />
                </div>
                <div className="modal-title">{t('This is NOT a quote.')}</div>
                <div className="modal-content">
                  <div className="mb-32 text-center">
                    {t(
                      'To receive a quote, you need to qualify for solar and we’ll need to contact you to get copies of your most recent electric bills.'
                    )}
                  </div>
                  <div className="mb-32">
                    <Checkbox onChange={this.handleCheck}>
                      {t('I understand that this is not a quote.')}
                    </Checkbox>
                  </div>
                  <div className="mb-32">
                    <Button
                      full
                      onClick={this.handleCloseDialog}
                      disabled={!checked}
                    >
                      {t('Continue')}
                    </Button>
                  </div>
                </div>
              </Modal>
            )}
            <Footer
              customText={
                !loading && (
                  <div className="custom-text">
                    <Trans
                      t={t}
                      i18nKey="DISCLAMER: <1/> While every effort has been taken to ensure that the calculations are as accurate as possible, the information provided by the calculator trades some accuracy for its simplicity and ease of use. The information is general in nature and is not intended to be a substitute for legal, financial, or electrical engineering advice. Please use the figures provided by the calculator as ballpark figure ONLY. For more accurate financial savings estimates, please reach out to us directly to have a system customized to the roof of your home. <1/> <1/> {{disclosure}} <1/> <1/> **Based on estimated system size, actual amount may vary."
                    >
                      DISCLAMER: <br />
                      While every effort has been taken to ensure that the
                      calculations are as accurate as possible, the information
                      provided by the calculator trades some accuracy for its
                      simplicity and ease of use. The information is general in
                      nature and is not intended to be a substitute for legal,
                      financial, or electrical engineering advice. Please use
                      the figures provided by the calculator as ballpark figure
                      ONLY. For more accurate financial savings estimates,
                      please reach out to us directly to have a system
                      customized to the roof of your home.
                      <br /> <br />
                      {{ disclosure }}
                      <br /> <br />
                      **Based on estimated system size, actual amount may vary.
                    </Trans>
                  </div>
                )
              }
            />
          </div>
          {company_name === 'The Pro Companies' && (
            // eslint-disable-next-line jsx-a11y/iframe-has-title
            <iframe
              src={`https://track.clickbooth.com/l/con?cbiframe=1&oid=67899&cbtid=${requestId}`}
              scrolling="no"
              frameBorder="0"
              width="1"
              height="1"
              loading="eager"
            ></iframe>
          )}
        </div>
      </>
    );
  }
}

const mapStateToProps = ({ user: { data }, ui: { theme } }) => ({
  user: data,
  theme,
});

const mapDispatchToProps = (dispatch) => ({
  actions: bindActionCreators(
    {
      setUserData,
      saveSession,
      navigateTo,
      trackForwardProgress,
    },
    dispatch
  ),
});

export default withTranslation('translation')(
  connect(mapStateToProps, mapDispatchToProps)(Proposals)
);
